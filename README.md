# DID Method Specification - did:earth

## Abstract
Interchain Identifiers (IIDs) are a family of Decentralized Identifier methods which are purpose-designed to identify, refer to, and interact with digital assets within blockchain namespaces.

The IID specification builds on the Decentralized Identifier (DID) core specification from the World Wide Web Consortium (W3C) [[1]](#ref1). IIDs are fully conformant DIDs and therefore are DIDs. IID Documents are DID documents. Unlike DIDs, IIDs only reference on-chain assets—which we will refer to as tokens—but should be taken as any type of tokens, such as NFTs, fungible tokens, tokenized namespace records, or other on-chain assets.

By restricting IIDs to on-chain assets, we create a new class of identifier uniquely suited to the requirements of tokens and other chain-native components. IID methods can be developed for any compatible blockchain, making them suitable for interoperable representations of tokens (and the cryptography that secures those tokens) regardless of the underlying chain.

IIDs also introduce a few new features—conformant extensions to the DID Core specification—that provide for privacy-respecting options for the full range of expected token functionality, including Linked Resources, On-chain Service Endpoints, and Accorded Rights.

DID Methods which conform to the IID specification resolve to DID Document representing how to securely interact with a uniquely identified digital asset, within a unique blockchain namespace. Because IIDs are DIDs, software applications that are conformant with the W3C specification will be able to inter-operate with IIDs and IID documents, although some IID-specific features may require additional tooling.


### DID Method Name
The namestring that shall identify this DID `method-name` is: `earth`.

A DID that uses this method MUST begin with the following prefix `did:earth`. The prefix string MUST be in lowercase. The remainder of the DID after the prefix, is specified below:

#### Method Specific Identifier
The DID `earth` method-specific identifier (`method-specific-id`) is made up of a `chainspace` and a `namespace` component. 

The `chainspace` is defined as a string that identifies a specific Cosmos blockchain (e.g. "ixo", "regen", "cosmos") where the DID reference is stored.

The `namespace` is defined as an alphanumeric string that identifies a specific Cosmos module in the `chainspace` (e.g. "nft", "bank", "staking") where the DID reference is stored. 

A chainspace is mandatory and MUST be included as a refence pointer to a specific Cosmos blockchain.

A namespace is optional and MAY be included as a reference pointer to a specific Cosmos blockchain network. If namespace is omitted it will default to the production network for a specified `chainspace` for the specific Cosmos blockchain.

**Not sure what crypto algorithm the method will use**
A did:earth DID must be unique by having the unique-id component be derived from the initial public key of the DID. For an Ed25519 public key, the first 16 bytes of the base-58 representation of the 256-bit public key is used to generate the unique-id.

Add Secp256k1 and Ed25519 suites.

[Cosmos Chain Registry](https://github.com/cosmos/chain-registry) will be used as the source of valid `chainspace` and `namespace` values. Every Cosmos blockchain consist of a genesis file that define the operational parameters including the `chain-id`. The `chain-id` MUST be unique (up to 50 characters), alphanumeric value across all Cosmos blockchains. For each Cosmos blockchain there will be an entry in the Cosmos Chain Registry that includes a file called `chain.json`. The `chain_name` will be used to represent the `chainspace` and the `chain_id` will be used to represent the `namespace` in the earth DID method. These values uniquely identify which Cosmos blockchain and network is targeted. 

https://w3c.github.io/did-core/#did-syntax

#### earth DID method syntax
```abnf
earth-did       = "did:earth:" chainspace ":" namespace ":" unique-id
chainspace      = 1*50namespace-char
chainspace-char = ALPHA / DIGIT
namespace       = 1*50idchar 
unique-id       = 1*255idchar
idchar          = ALPHA / DIGIT / "." / "-" / "_" / pct-encoded

```
NOTE: Check on "." for idchar and double chain name limits

#### Examples of `did:earth` identifiers

A DID written to the IXO "mainnet" ledger `namespace`:

```abnf
did:earth:ixo:impacthub-3:7Tqg6BwSSWapxgUDm9KKgg
```

A DID written to the IXO "mainnet" ledger `namespace`:

```abnf
did:earth:ixo:7Tqg6BwSSWapxgUDm9KKgg
```

A DID written to the IXO "testnet" ledger `namespace`:

```abnf
did:earth:ixo:pandora-4:6cgbu8ZPoWTnR5Rv5JcSMB
```

A DID written to the Regen "mainnet" ledger `namespace`:

```abnf
did:earth:regen:testnet:6cgbu8ZPoWTnR5Rv5JcSMB
```

A DID written to the IXO "mainnet" ledger using a specific Cosmos module `namespace`:

```abnf
did:earth:ixo:impacthub-3:nft:6cgbu8ZPoWTnR5Rv5JcSMB
```

A DID written to the IXO "mainnet" ledger using a specific Cosmos module `namespace`:

```abnf
did:earth:ixo:nft:6cgbu8ZPoWTnR5Rv5JcSMB
```

**Playing around with the concept of `path`, `query` and `fragments` related to specific Cosmos modules

A DID written to the IXO "mainnet" ledger using a specific Cosmos module `namespace`:

```abnf
did:earth:ixo:impacthub-3?nft#6cgbu8ZPoWTnR5Rv5JcSMB
```

### DID Documents (DIDDocs)
A DID Document ("DIDDoc") associated with a earth DID is a set of data describing a DID subject. The [representation of a DIDDoc when requested for production](https://www.w3.org/TR/did-core/#representations) MUST meet the DID Core specifications.

### Linked Resources

The `linkedResource` property provides a privacy-enabled way to attach
digital resources to an on-chain asset. This is an optional property which
may contain one or more resource descriptors in array. This property provides the metadata required for accessing and using the specified resource, such as the type of resource, a proof to verify the resource, and a service endpoint for requesting and retrieving the
resource.

Resources may be provided in-line or by secure reference. Inline resources are appropriate only for use cases that need to directly include the resource in the IID Document. In many cases, this is a privacy problem. However, for some use cases, resources must be specified for on-chain execution, which justifies the added bytes and potential disclosure risk. The resource descriptor provides for a flexible representation of various mime types, compression, and encoding, as required for the use.

This approach allows token owners to manage privacy in three key ways:

1.  Avoids posting potentially sensitive information on-chain in an unavoidably public and irrevocable manner.
2.  Provides a service endpoint that can apply appropriate privacy and security checks before revealing information.
3.  The hashgraph resource descriptor type obscures not only the content of the linked resource, but also the quantity of resource objects.

Resources may be secured by specifying a `proofType` of hash or hashgraph. A hashgraph uses a merkle tree of hashes for external content associated with this asset. A resource descriptor of this type obscures both the type and the number of such resources, while allowing each such resource to be verifiably linked to the asset. It also provides for privacy-respecting verification of complete disclosure. Anyone who needs to prove they have all of the linked resources can compare their own hash graph of resources with the value stored in the IID Document. Note this anti-censorship technique requires a verifier to discover the type and nature of those resources on their own.

Proposed properties for resource descriptors in the `LinkedResource` property:
```javascript
{
	"linkedResource": [{

		"path"(optional): // IID Resource path for this resource in the asset namespace, e.g., /myResource.png

			"id"(optional): // IID Reference ID for this resource, e.g., #myResource.png

			"rel"(optional): // the relationship of this resource to the IID Asset

			"type": "nft:ResourceDescriptor", // The JSON-LD type of this resource

		"proof": [ // an array of proofs, any of which may be used
			{
				"type"(optional): "hash" | "hashgraph" | "hashset" // the form of proof used to verify the resource as authentic
				"stage"(optional): "raw" | "compressed" | "encrypted" | "encoded" // the 
				"value"(optional): hash | hashgraph | hashset, // the actual proof for this particular resource
			}
		],
		"resourceFormat": media type, // the IANA media-type of the linked resource

		"compression"(optional): "gzip" | "none", // the compression (performed on the raw resource) of an inline resource

		"encryption": [open ended
			for arbitrary extensibility
		], // the encryption technique applied after compression and before encoding

		"encoding"(optional): "native" | "multibase" | "string", // the encoding (after compression and encryption) of an inline resource; "native" means the inline resource is native JSON-LD with neither compression nor encryption

		"endpoint"(optional): url, // a URL at which this resource can be retrieved before decrypting and decrompressing

		"resource"(optional): "..."
		a representation of the resource
		for inline
		distribution,
		encoded as per "encoding",
		then compre

	}]
}
```

#### Elements for a W3C specification compliant DIDDoc representation

1. **`id`**: Target DID as base58-encoded string for 16 or 32 byte DID value with earth DID Method prefix `did:earth:<chainspace>:<namespace>:`.
2. **`controller`** (optional): A list of fully qualified DID strings or one string. Contains one or more DIDs who can update this DIDdoc. All DIDs must exist.
3. **`verificationMethod`** (optional): A list of Verification Methods
4. **`authentication`** (optional): A list of strings with key aliases or IDs
5. **`assertionMethod`** (optional): A list of strings with key aliases or IDs
6. **`capabilityInvocation`** (optional): A list of strings with key aliases or IDs
7. **`capabilityDelegation`** (optional): A list of strings with key aliases or IDs
8. **`keyAgreement`** (optional): A list of strings with key aliases or IDs
9. **`service`** (optional): A set of Service Endpoint maps
10. **`alsoKnownAs`** (optional): A list of strings. A DID subject can have multiple identifiers for different purposes, or at different times. The assertion that two or more DIDs refer to the same DID subject can be made using the `alsoKnownAs` property.
11. **`@context`** (mandatory): A list of strings with links or JSONs for describing specifications that this DID Document is following to.

```
Must be JSON-LD
https://w3c.github.io/did-core/#json-ld

The DID document, DID document data structures, and representation-specific entries map MUST be serialized to the JSON-LD representation according to the JSON representation production rules as defined in § 6.2 JSON.

In addition to using the JSON representation production rules, JSON-LD production MUST include the representation-specific @context entry. The serialized value of @context MUST be the JSON String https://www.w3.org/ns/did/v1, or a JSON Array where the first item is the JSON String https://www.w3.org/ns/did/v1 and the subsequent items are serialized according to the JSON representation production rules.
```

#### Additonal elements specific to the Earth DID method DIDDoc representation
13. **`linkedResource`** (optional): is a new IID document property for specifying how to verify, and optionally retrieve, any and all resources necessary for the proper functioning of the on-chain asset. Like email attachments, LinkedResources provide for attaching arbitrary media to an onchain asset.
14. **`transclude`** (optional): is a new IID document property for specifying where in an IID document to transclude a linked resource. If present, the value of this property MUST be one (or an array of more than one) Linked Resources that eventually dereferences to a raw JSON-LD object. The properties of that JSON-LD object will be injected into the current IID document, replacing the transclude property entirely. The properties of the transcluded JSON-LD MUST be transformed to their absolute representation using the object's `@context` value prior to transclusion. The associated linked resources MUST have a `rel` value of "extension" and a `mediaType` value of "application/ld-json"
15. **`extension`** (optional): a type of Linked Resource. A JSON-LD extension of the current document. The RDF statements in the extension are to be included in the current IID document, where specified by a "transclude" property. For example, additional service endpoint definitions may be added in a linked resource. These endpoints can be verified as being associated with the IID. But only by those parties who secure the definitions through other privacy respecting mechanisms. This property standardizes how to verifiably move arbitrary RDF statements outside of the IID document context, to provide additional security and privacy.
16. **`executableRight`** (optional): a type of Linked Resource. This resource is a machine-executable capability that can be invoked by the IID owner or its delegate, using cryptographic materials defined elsewhere in the IID document Verification Methods property.
17. **`assertion`** (optional): a rel value for a Linked Resource. Verifiable credentials, verified claims, claim tokens as described in NFT-RFC-008. This allows arbitrary, yet verifiable attestations to be made either about the asset or about the resources defined by IID references. The attributes represented in these claims can be retrieved through the token interface using a Query by Example. (graph query) mechanism.
18. **`rel`** (optional): a property of Linked Resource. Defines the relationship of this resource to the IID asset. Known values include:
     1. "evidence" -- The resource is evidence for the creation of the asset.
     2. "encodedRepresentation" -- The resource is a binary representation of the asset, interpretable by compatible software to display or interact with.
     3. "visualRepresentation" -- The resource is a visual representation of the asset
19. **`accordedRight`** (optional): a type of relationship to a Linked Resource. Specifies the rights accorded to the IID owner, or its deligate, which may be executed by physical-world institutions or processes. Such as a digital driver's license according certain rights to drive, or a theater ticket according access to a show. The representation framework for such rights must be non-prescriptive, including both plain text statements of rights, e.g., "The controller of this IID is entitled to ...", or more rigorous and computationally evaluatable RDF statements, which might describe in great detail a range of benefits that accompany the basic rights of the token.
20. **`displayName`** (optional): a property of Linked Resources that provide a brief textual label for display.
21. **`displayDescription`** (optional): a property of Linked Resources that provide a longer text phrase for displaying additional detail about the asset.
22. **`displayIcon`** (optional): a property of Linked Resources that provide a URL for an image asset to use when displaying the asset.

##### Example of DIDDoc representation

```jsonc
{
	"@context": [
		"https://www.w3.org/ns/did/v1",
	     "https://w3id.org/security/suites/ed25519-2020/v1"
      "http://w3id.org/earth/ns/v1"]
	],
	"id": "did:earth:ixo:mainnet:1234567",
	"verificationMethod": [{
			"id": "did:earth:ixo:mainnet:1234567#authKey1",
			"type": "Ed25519VerificationKey2020", // external (property value)
			"controller": "did:earth:ixo:mainnet:1234567",
			"publicKeyMultibase": "zAKJP3f7BD6W4iWEQ9jwndVTCBq8ua2Utt8EEjJ6Vxsf"
		},
		{
        	"id": "did:earth:ixo:mainnet:1234567#capabilityInvocationKey",
                "type": "Ed25519VerificationKey2020", // external (property value)
                "controller": "did:earth:ixo:mainnet:1234567",
                "publicKeyMultibase": "z4BWwfeqdp1obQptLLMvPNgBw48p7og1ie6Hf9p5nTpNN"
		}
	],
	"authentication": ["did:earth:ixo:mainnet:1234567#authKey1"],
	"capabilityInvocation": ["did:earth:ixo:mainnet:1234567#capabilityInvocationKey"],
	{
	"linkedResource": [{
			"path": "/myResource.png",
			"id": "#myResource.png",
			"rel": ,
			"type": "nft:ResourceDescriptor",
			"proof": [
			{
				"type": "hash" | "hashgraph",
				"stage": "raw" | "compressed" | "encrypted" | "encoded",
				"value": "hash" | "hashgraph" 
			}
			],
			"resourceFormat": "application/json",
			"compression": "gzip" | "none",
			"encryption": [open ended for arbitrary extensibility],
			"encoding": "native" | "multibase" | "string",
			"endpoint": "http://www.example.com/myResource",
			"resource": "..."
		}]
	}
}
```

#### State format for DIDDocs on ledger

#### DIDDoc metadata

##### Example of DIDDoc metadata

#### Verification method

Verification methods are used to define how to authenticate / authorise interactions with a DID subject or delegates. Verification method is an OPTIONAL property.

1. **`id`** (string): A string with format `did:ixo:<chainspace>:<namespace>#<key-alias>`
2. **`controller`**: A string with fully qualified DID. DID must exist.
3. **`type`** (string)
4. **`publicKeyJwk`** (`map[string,string]`, optional): A map representing a JSON Web Key that conforms to [RFC7517](https://tools.ietf.org/html/rfc7517). See definition of `publicKeyJwk` for additional constraints.
5. **`publicKeyMultibase`** (optional): A base58-encoded string that conforms to a [MULTIBASE](https://datatracker.ietf.org/doc/html/draft-multiformats-multibase-03)
encoded public key.

**Note**: Verification method cannot contain both `publicKeyJwk` and `publicKeyMultibase` but must contain at least one of them.

##### Example of Verification method in a DIDDoc

```jsonc
{
  "id": "did:ixo:mainnet:123456#key-0",
  "type": "JsonWebKey2020",
  "controller": "did:ixo:mainnet:123456",
  "publicKeyJwk": {
    "kty": "OKP",
    // external (property name)
    "crv": "Ed25519",
    // external (property name)
    "x": "VCpo2LMLhn6iWku8MKvSLg2ZAoC-nlOyPVQaO3FxVeQ"
    // external (property name)
  }
}
```
#### Service
Services can be defined in a DIDDoc to express means of communicating with the DID subject or associated entities.

1. **`id`** (string): The value of the `id` property for a Service MUST be a URI conforming to [RFC3986](https://www.rfc-editor.org/rfc/rfc3986). A conforming producer MUST NOT produce multiple service entries with the same ID. A conforming consumer MUST produce an error if it detects multiple service entries with the same ID. It has a follow formats: `<DIDDoc-id>#<service-alias>` or `#<service-alias>`.
2. **`type`** (string): The service type and its associated properties SHOULD be registered in the [DID Specification Registries](https://www.w3.org/TR/did-spec-registries/)
3. **`serviceEndpoint`** (strings): A string that conforms to the rules of [RFC3986](https://www.rfc-editor.org/rfc/rfc3986) for URIs, a map, or a set composed of a one or more strings that conform to the rules of
[RFC3986](https://www.rfc-editor.org/rfc/rfc3986) for URIs and/or maps.

##### Example of Service in a DIDDoc

```jsonc
{
  "id":"did:ixo:mainnet:123456#linked-domain",
  "type": "LinkedDomains",
  "serviceEndpoint": "https://bar.example.com"
}
```



### DID transaction operations
DID and associated documents are managed by a Cosmos-SDK module that uses the gRPC communication protocol. See [method specification](https://hackmd.io/1Nh-r80_SiyKvWzotvkTSQ) for details on how the create, read, update and delete (CRUD) operations are handed in the Cosmos IID module. 

#### Create DID (Register)
This operation creates a new DID using the did:earth method along with associated DID Document representation.

To create and publish a DID document use the message 

```golang
MsgCreateIdentifier(creator string, IidDocument string)
```

```golang
MsgCreateIdentifier(method-specific-id string, namespace string)
```

```golang
MsgCreateRemoteIdentifier(method-specific-id string, chainspace string, namespace string)
```

The creators public key MUST be extracted from the IidDocument.

If the input DID is not a valid DID for the Cosmos method, or if the DID already exists on-chain, the message returns an error.


- [ ] What about multiple signatures should the on-chain asset be owned/controlled by multiple parties?

- **`creator`**: `MsgCreateIdentifier` should be a fully qualified DID of type `did:earth:<chainspace>:<namespace>`. Allowed `chainspace` and `namespace` values are available in the [Cosmos Chain Registry](https://github.com/cosmos/chain-registry)
- **`IidDocument`**: MUST be a compliant IidDocument according to the DID method specification.



- [ ] What about the optional parameters required by DID core as per below?

- **`controller, verificationMethod, authentication, assertionMethod, capabilityInvocation, capabilityDelegation, keyAgreement, service, alsoKnownAs, context`**: Optional parameters in accordance with DID Core specification properties.

##### Client request format for create DID
```jsonc
WriteRequest (MsgCreateIdentifier(creator string, IidDocument string)
```

##### Example of a create DID client request
```jsonc
WriteRequest{
        "data": MsgCreateIdentifier {
                "id": "did:earth:ixo:mainnet:1234567",
                "IidDocument": {
                        "id": "did:earth:ixo:mainnet:1234567",
                        "verificationMethod": [{
                                        "id": "did:earth:ixo:mainnet:1234567#authKey1",
                                        "type": "Ed25519VerificationKey2020", // external (property value)
                                        "controller": "did:earth:ixo:mainnet:N22N22KY2Dyvmuu2PyyqSFKue",
                                        "publicKeyMultibase": "zAKJP3f7BD6W4iWEQ9jwndVTCBq8ua2Utt8EEjJ6Vxsf"
                                },
                                {
                                        "id": "did:earth:ixo:mainnet:1234567#capabilityInvocationKey",
                                        "type": "Ed25519VerificationKey2020", // external (property value)
                                        "controller": "did:earth:ixo:mainnet:N22N22KY2Dyvmuu2PyyqSFKue",
                                        "publicKeyMultibase": "z4BWwfeqdp1obQptLLMvPNgBw48p7og1ie6Hf9p5nTpNN"
                                }
                        ],
                        "authentication": ["did:earth:ixo:mainnet:1234567#authKey1"],
                        "capabilityInvocation": ["did:earth:ixo:mainnet:1234567#capabilityInvocationKey"],
                        "signatures": {
                                "Verification Method URI": "<signature>"
                                // Multiple verification methods and corresponding signatures can be added here
                        }
                }
        }
}
```

#### Read DID (Resolve and Verify)
To resolve DID earth method DID Documents the `QueryIdentifierDocument` operation to fetch a response from the ledger. The integrity of the DID documents stored on the ledger is guaranteed by the underlying blockchain protocol. DID resolution requests can be sent to the gRPC IID interface for a node by passing the fully-qualified DID. 

A DID can be resolved using the gRPC message:
```golang
QueryIdentifierDocument(id string)
```
The operation CAN be executed by anyone and is publicly available.

- **`id`**: `QueryIdentifierDocument` should be a fully qualified DID of type `did:earth:<chainspace>:<namespace>`. It MUST be the DID that is to be resolved. Allowed `chainspace` and `namespace` values are available in the [Cosmos Chain Registry](https://github.com/cosmos/chain-registry)


- [ ] What about metadata?
- **`metadata`**: Contains DIDDoc metadata? `created`, `updated`, `valid`, `versionId`

The operation MUST return the DID Document and metadata.

##### Client request format to resolve a DID to it's DID Document
```jsonc
WriteRequest QueryIdentifierDocument(id string)
```
##### Example of DID resolution to DID Document client request
```jsonc
WriteRequest{
        "data": MsgUpdateIidDocument {
                "id": "did:earth:ixo:mainnet:1234567"
        }
}
```
#### Update DID
This operation allow updates to DID Documents by the controller(s).

Please note that the DID will remain the same, but the contents of the DID document could change, e.g., by including a new verification key or adding service endpoints.

A DID can be updated using the gRPC message:
```golang
MsgUpdateIidDocument(id string, controller string, identifiers list, verificationMethods list, verificationRelationships list, service service, linkedResources list, accordedRights list)
```
The operation MUST be executed by an authorized controller of the DID.

- **`id`**: `MsgUpdateIidDocument` should be a fully qualified DID of type `did:earth:<chainspace>:<namespace>`. It MUST be the DID that is to be deleted. Allowed `chainspace` and `namespace` values are available in the [Cosmos Chain Registry](https://github.com/cosmos/chain-registry)
- **`controller`**: should be a fully qualified DID of type `did:earth:<chainspace>:<namespace>`.
- **`identifiers, verificationMethods, verificationRelationships, service, linkedResources, accordedRights`**: Optional parameters in accordance with DID Core and IID specification properties.

The operation MUST update the DID Document and metadata. The operation is not reversible.

##### Client request format to update a DID Document
```jsonc
WriteRequest MsgUpdateIidDocument(id string, controller string, identifiers list, verificationMethods list, verificationRelationships list, service service, linkedResources list, accordedRights list)
```
##### Example of update DID Document client request
```jsonc
WriteRequest{
        "data": MsgUpdateIidDocument {
                "id": "did:earth:ixo:mainnet:1234567",
        "controller": "did:earth:ixo:mainnet:1234567"
        "identifiers": [],
        "verificationMethods": [],
        "verificationRelationships": [],
        "service": service,
        "linkedResources": [],
        "accordedRights": []
        }
}
```


#### Revoke DID
This operation deactivate or delete DID records using the did:earth method.
#### Deactive DID

A DID can be deactivated using the gRPC message:
The operation MUST be executed by an authorized controller of the DID.

```golang
MsgDeactivateIdentifier(id string, Controller string)
```

- **`id`**: `MsgDeactivateIdentifier` should be a fully qualified DID of type `did:earth:<chainspace>:<namespace>`. It MUST be the DID that is to be deactivated. Allowed `chainspace` and `namespace` values are available in the [Cosmos Chain Registry](https://github.com/cosmos/chain-registry)
- **`controller`**: should be a fully qualified DID of type `did:earth:<chainspace>:<namespace>`.

The operation MUST update the DID document metadata and set the Active value to False. The operation is not reversible.

##### Client request format to deactivate a DID
```jsonc
WriteRequest MsgDeactivateIdentifier(id string, Controller string)
```

A DID can be deleted using the gRPC message:
The operation MUST be executed by an authorized controller of the DID.
```golang
MsgDeactivateIdentifier(id string, Controller string)
```
##### Example of deactivate DID client request
```jsonc
WriteRequest{
        "data": MsgDeactivateIdentifier {
                "id": "did:earth:ixo:mainnet:1234567",
        "controller": "did:earth:ixo:mainnet:1234567"
        }
}
```

#### Delete DID
A DID can be deleted using the gRPC message:
The operation MUST be executed by an authorized controller of the DID.

- **`id`**: `MsgDeleteIdentifier` should be a fully qualified DID of type `did:earth:<chainspace>:<namespace>`. It MUST be the DID that is to be deleted. Allowed `chainspace` and `namespace` values are available in the [Cosmos Chain Registry](https://github.com/cosmos/chain-registry)
- **`controller`**: should be a fully qualified DID of type `did:earth:<chainspace>:<namespace>`.

The operation MUST delete the DID and the DID document and metadata. The operation is not reversible.

##### Client request format to delete a DID
```jsonc
WriteRequest MsgDeleteIdentifier(id string, Controller string)
```
##### Example of delete DID client request
```jsonc
WriteRequest{
        "data": MsgDeleteIdentifier {
                "id": "did:earth:ixo:mainnet:1234567",
        "controller": "did:earth:ixo:mainnet:1234567"
        }
}
```
### Security Considerations

### Privacy Considerations

```jsonc
    "proof": [{
            "type": "hash" | "hashgraph" | "hashset",
            "stage": "raw" | "compressed" | "encrypted" | "encoded",
            "value": "hash" | "hashgraph" | "hashset"
            }]
```
#### Hash based Linked Resource

#### Hash Graph based Linked Resource

#### Hash Set based Linked Resource
The `linkedResource` property provides a privacy-enabled way to attach
digital resources to an on-chain asset. This is an optional property which
may contain one or more resource descriptors in array. This property provides the metadata required for accessing and using the specified resource, such as the type of resource, a proof to verify the resource, and a service endpoint for requesting and retrieving the resource.

Resources may be provided in-line or by secure reference. Inline resources are appropriate only for use cases that need to directly include the resource in the IID Document. In many cases, this is a privacy problem. However, for some use cases, resources must be specified for on-chain execution, which justifies the added bytes and potential disclosure risk. The resource descriptor provides for a flexible representation of various mime types, compression, and encoding, as required for the use.

This approach allows token owners to manage privacy in three key ways:

1.  Avoids posting potentially sensitive information on-chain in an unavoidably public and irrevocable manner.
2.  Provides a service endpoint that can apply appropriate privacy and security checks before revealing information.
3.  The hashgraph resource descriptor type obscures not only the content of the linked resource, but also the quantity of resource objects.

Resources may be secured by specifying a `proofType` of hash or hashgraph. A hashgraph uses a merkle tree of hashes for external content associated with this asset. A resource descriptor of this type obscures both the type and the number of such resources, while allowing each such resource to be verifiably linked to the asset. It also provides for privacy-respecting verification of complete disclosure. Anyone who needs to prove they have all of the linked resources can compare their own hash graph of resources with the value stored in the IID Document. Note this anti-censorship technique requires a verifier to discover the type and nature of those resources on their own.
                    
## References
<a name="ref1">[1]</a> Decentralized Identifiers (DIDs) v1.0. World Wide Web Consortium.
Online at
[[https://www.w3.org/TR/did-core/]](https://www.w3.org/TR/did-core/).
Accessed February 15, 2021.



# Questions for Shaun

1. Is did:earth going to support *any* cosmos chain, likely by using the chain registry? https://github.com/cosmos/chain-registry/blob/master/comdex/chain.json
https://github.com/cosmos/chain-registry

2. Can we get cosmos to add testnet entries to the chain registry. If we don't do that, we don't have a way to have testnet-based DIDs.

FWIW, I think this is the best way to solve this problem (adding testnet entries to the chain registry) all we need is a unique name for all chain/network combinations


### Notes
*did:earth should just exist on IXO chain*
*did:earth is an iid method can be created on any chain*