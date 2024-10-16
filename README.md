# Merkle Root Extension Specification

- **Title:** Merkle Root
- **Identifier:** <https://stac-extensions.github.io/merkle/v1.0.0/schema.json>
- **Field Name Prefix:** `merkle`
- **Scope:** Item, Collection, Catalog
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @jonhealy1
  
This extension specifies a way to ensure metadata integrity for STAC Items, Collections and catalogs by encoding them in a Merkle Tree via hashing. Items are hashed using a hash function (e.g., SHA-256), and this hash is stored as a field in an item's properties. Details concerning the methods used for hashing are stored in a separate object. To produce the Merkle root identifier for a Collection, the hash from every Item is taken into account. This process ensures the integrity of STAC Items and Collections. Additionally, where multiple Collections make up a Catalog, the Merkle Root identifiers from each Collection can be used to create a Merkle root identifier for the whole Catalog.  

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [x] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [ ] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name           | Type                               | Description                                  |
| -------------------- | ---------------------------------- | -------------------------------------------- |
| `merkle:item_hash`     | string                             | **REQUIRED**. (in Items). A cryptographic hash of the Item's metadata, used to verify the integrity of the Item.
| `merkle:hash_method`   | [Hash Method Object](#hash-method-object) | **REQUIRED**. (in Items and Collections). An object describing the method used to compute `merkle:item_hash` or `merkle:root`, including the hash function, fields included, and ordering.                        |
| `merkle:root`          | string                             | **REQUIRED**. (in Collections and Catalogs). The Merkle root hash representing the Collection or Catalog, used to verify the integrity of all Items and sub-Collections/Catalogs.                      |

### Additional Field Information

#### merkle:item_hash

- **Type:** string
- **Description:** A cryptographic hash of the Item's metadata, computed according to the method specified in `merkle:hash_method`. This hash allows users to verify that the Item's metadata has not been altered.

#### merkle:hash_method

- **Type:** Hash Method Object
- **Description:** An object that specifies how `merkle:item_hash` (in Items) or `merkle:root` (in Collections and Catalogs) was computed, including the hash function used, the fields included, ordering, and any special considerations. This provides transparency and allows users to accurately verify the hash.

#### merkle:root

- **Type:** string
- **Description:** The Merkle root hash representing the Collection or Catalog. It is computed by building a Merkle tree from the `merkle:item_hash` values (for Collections) or from the `merkle:root` values of child Collections and Catalogs (for Catalogs). This root hash provides a single value that represents the integrity of all underlying Items and Collections.

### Hash Method Object

The `merkle:hash_method` object provides details about the hash computation method used for `merkle:item_hash`.

| Field Name     | Type   | Description                                  |
| -------------- | ------ | -------------------------------------------- |
| `function`         | string | **REQUIRED**. The cryptographic hash function used (e.g., `sha256`, `sha3-256`). 
| `fields`          | [string] | **REQUIRED** (for Items). An array of fields included in the hash computation. Use `"*"` or `"all"` to indicate that all fields are included. For nested fields, dot notation should be used (e.g., `properties.datetime`, `assets.image`). 
| `ordering`         | string | **REQUIRED** (for Collections). Describes how the hashes are ordered when building the Merkle tree (e.g., "ascending by merkle:item_hash value"). 
| `description`          | string | Optional. Additional details or notes about the hash computation method, such as serialization format or any special considerations. |

## Computing Hashes and Merkle Roots

### Computing `merkle:item_hash`
1. Prepare Metadata:
   - Include all fields specified in `merkle:hash_method.fields`.
   - If `fields` is "*" or "all", include all fields of the item.
2. Serialize Metadata:
   - Use canonical JSON serialization with sorted keys and consistent formatting.
   - Ensure consistent encoding (e.g., UTF-8).
3. Compute Hash:
   - Apply the specified cryptographic hash function (e.g., SHA-256) to the serialized metadata.

### Computing `merkle:root` for Collections and Catalogs
1. Collect Hashes:
   - For Collections: Gather `merkle:item_hash` values from all Items.
   - For Catalogs: Gather `merkle:root` values from all child Collections and Catalogs.
2. Order Hashes:
   - Order the hashes according to the method specified in `merkle:hash_method.ordering`.
3. Build Merkle Tree:
   - Pairwise hash the ordered hashes, proceeding up the tree until a single hash remains — the `merkle:root`.
4. Include `merkle:hash_method`:
   - Specify the method used in the Collection's or Catalog's `merkle:hash_method` field, including any details necessary for users to replicate the process.

## Examples
### Item Example
```jsonc
{
  "type": "Feature",
  "stac_version": "1.1.0",
  "id": "item-001",
  "properties": {
    "datetime": "2024-10-15T12:00:00Z",
    "merkle:item_hash": "3a7bd3e2360a8e7d9f5b1c2d4e6f7890abcdef1234567890abcdef1234567890",
    "merkle:hash_method": {
      "function": "sha256",
      "fields": ["*"],
      "description": "Computed using canonical JSON serialization of all fields."
    }
    // ... other properties
  },
  "geometry": {
    // ... geometry definition
  },
  "links": [
    // ... item links
  ],
  "assets": {
    // ... item assets
  }
}

```

### Collection Example
```jsonc
{
  "type": "Collection",
  "stac_version": "1.1.0",
  "id": "collection-123",
  "description": "Sample Collection with Merkle Root",
  "properties": {
    "merkle:root": "abc123def4567890abcdef1234567890abcdef1234567890abcdef1234567890",
    "merkle:hash_method": {
      "function": "sha256",
      "ordering": "ascending",
      "description": "Computed by ordering `merkle:item_hash` values in ascending order and building the Merkle tree."
    }
    // ... other properties
  },
  "extent": {
    // ... spatial and temporal extent
  },
  "links": [
    // ... collection links
  ],
  "license": "proprietary"
}
```

### Catalog Example
```jsonc
{
  "type": "Catalog",
  "stac_version": "1.1.0",
  "id": "catalog-001",
  "description": "Sample Catalog with Merkle Root",
  "properties": {
    "merkle:root": "f1e2d3c4b5a67890abcdef1234567890abcdef1234567890abcdef1234567890",
    "merkle:hash_method": {
      "function": "sha256",
      "ordering": "ascending",
      "description": "Computed by ordering `merkle:root` values of child Collections in ascending order and building the Merkle tree."
    }
  },
  "links": [
    {
      "rel": "child",
      "href": "collection-123.json"
    },
    {
      "rel": "child",
      "href": "collection-456.json"
    }
  ]
}
```

## Relation types

This extension does not introduce any new relation types. The standard STAC relation types should be used as applicable in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md). Instructions
for running tests are copied here for convenience.

**Note:** This extension is currently a proposal and is open for feedback from the STAC community. Your input is valuable to refine and adopt the Merkle Root Extension.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
