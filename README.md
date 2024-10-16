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
| `merkle:item_hash`     | string                             | **REQUIRED**. A cryptographic hash of the Item's metadata, used to verify the integrity of the Item.
| `merkle:hash_method`   | [Hash Method Object](#hash-object) | **REQUIRED**. An object describing the method used to compute merkle:item_hash, including the hash function and the fields included in the computation.                        |
| `merkle:root`          | string                             | **REQUIRED**. The Merkle root hash representing the Collection or Catalog, used to verify the integrity of all Items and sub-Collections/Catalogs.                      |

### Additional Field Information

#### merkle:item_hash

- **Type:** string
- **Description:** A cryptographic hash of the Item's metadata, computed according to the method specified in `merkle:hash_method`. This hash allows users to verify that the Item's metadata has not been altered.

#### merkle:hash_method

- **Type:** Hash Method Object
- **Description:** An object that specifies how `merkle:item_has`h was computed, including the hash function used and which fields were included. This provides transparency and allows users to accurately verify the hash.

#### merkle:root

- **Type:** string
- **Description:** The Merkle root hash representing the Collection or Catalog. It is computed by building a Merkle tree from the `merkle:item_hash` values (for Collections) or from the `merkle:root` values of child Collections and Catalogs (for Catalogs). This root hash provides a single value that represents the integrity of all underlying Items and Collections.

### XYZ Object

This is the introduction for the purpose and the content of the XYZ Object...

| Field Name | Type   | Description                                  |
| ---------- | ------ | -------------------------------------------- |
| x          | number | **REQUIRED**. Describe the required field... |
| y          | number | **REQUIRED**. Describe the required field... |
| z          | number | **REQUIRED**. Describe the required field... |

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type           | Description                           |
| -------------- | ------------------------------------- |
| fancy-rel-type | This link points to a fancy resource. |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

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
