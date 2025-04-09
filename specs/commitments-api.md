# Commitments API Specification

## Abstract
The Commitments API is a set of endpoints for issuing proposer commitments.

## Motivation
The proposer commitments space is rapidly evolving, with preconfirmations representing just one early example. By providing a standardized Commitments API for users / applications / wallets to interact with commitment issuers, we can establish a unified interface that scales as new protocols enter with novel commitment types. This approach aims to prevent fragmentation and keeps complexity manageable as the ecosystem grows.

## API Goals
- Support both L1 and L2 commitments
- Support *generic* proposer commitments rather than just preconfs
- Simple wallet integration
- Support for users to learn about capabilities / fees

## Terminology
- **Commitment**: A binding and objectively verifiable promise made about a chain.
- **Proposer Commitment**: A commitment made by an L1 proposer, e.g., I promise to include transaction with hash `0x1234...` in an L1 block during slot 1000. 
- **Gateway**: The party with committment-issuing rights granted by the Proposer.
- **Slasher**: A smart contract containing logic to arbitrate disputes and slash collateral should commitments be broken. 

## API Scope
**In Scope**
- Requesting a commitment to be made
- Requesting to see a previously signed commitment
- Requesting to see a the upcoming capabilities of a Gateway
- Requesting fee information for a specific commitment request

**Out of Scope**

The [Constraints Specs](https://github.com/eth-fabric/constraints-specs) and [API](https://eth-fabric.github.io/constraints-specs/).


# API Summary

| **Namespace** | **Method** | **Endpoint** | **Description** |
| --- | --- | --- | --- |
| `commitments`   | `POST` | [/commitments/v0/gateway/commitment](commitments-api.md#postcommitmentrequest)        | ... |
| `commitments`   | `GET` | [/commitments/v0/gateway/commitment](commitments-api.md#getsignedcommitment)        | ... |
| `commitments`   | `GET` | [/commitments/v0/gateway/slots](commitments-api.md#getslots)        | ... |
| `commitments`   | `POST` | [/commitments/v0/gateway/fee](commitments-api.md#getfeeinfo)        | ... |

# Schemas
### CommitmentRequest
```Python
# A CommitmentRequest message created by a user
class CommitmentRequest(Container):
    # Type of commitment being requested
    commitment_type: uint64
    # Opaque input bytes used as part of the commitment
    payload: Bytes
    # Slasher contract for resolving commitment disputes
    slasher: Address
```

### Commitment
```Python
# A Commitment message responding to a CommitmentRequest
class Commitment(Container):
    # The type of commitment being made
    commitment_type: uint64
    # Opaque payload bytes of the commitment
    payload: Bytes
    # Hash of the CommitmentRequest this Commitment is for
    request_hash: uint64
    # Slasher contract for resolving commitment disputes
    slasher: Address
```

### SignedCommitment
```Python
# A signed Commitment binding to a CommitmentRequest
class SignedCommitment(Container):
    # The commitment message that was signed
    commitment: Commitment
    # The signature of the commitment message
    signature: ECDSASignature
```

### SlotInfo
```Python
# Information about a Gateway's offerings at a specific slot
class SlotInfo(Container):
    # The L1 slot number 
    slot: Slot
    # The list of chain offerings
    offerings: list[Offering]
```

### SlotInfoResponse
```Python
# Repsonse containing multiple SlotInfo
class SlotInfoResponse(Container):
    # A list of slot infos
    slots: list[SlotInfo]
```

### Offering
```Python
# Specifies which commitments can be made for a specific chain
class Offering(Container):
    # The id of the target chain
    chain_id: uint64
    # The types of commitments offered for the target chain
    commitment_types: list[uint64]
```

### FeeInfo
```Python
# Response body for fee information
class FeeInfo(Container):
    # Opaque bytes containing fee info related to the commitment type
    payload: Bytes
    # Type of commitment being requested
    commitment_type: uint64
```

# Endpoints

### **postCommitmentRequest**

- **Method:** `POST /commitments/v0/gateway/commitment`
- **Response:** `SignedCommitment`
- **Headers:**
    - `Content-Type: application/json`
- **Body:** JSON object of type `CommitmentRequest`

- **Description**

    A `CommitmentRequest` contains an opaque `payload` bytes input that can be decoded according to the `commitment_type`. By making a request, the user / app / wallet is asking for the Gateway to make a commitment that is enforceable via the specified `slasher` contract.

    Each `commitment_type` has its own rules for how a Gateway maps a `CommitmentRequest.payload` to a `Commitment.payload`. The `Commitment.request_hash` field is used to bind the `Commitment` to a specific `CommitmentRequest`, however this is not required to correspond 1:1. In the [appendix](commitments-api.md#appendix), we show how a `Commitment` can correspond to multiple `CommitmentRequest` containers by chaining their hashes.

    The `SignedCommitment` response containts the ECDSA signature over the `Commitment`. The Commitments API doesn't specify the Gateway's key, but when used in conjunction with the Constraints API, the expectation is the key of the Gateway's `committer` address is used.

---

### **getSignedCommitment**

- **Method:** `GET /commitments/v0/gateway/commitment/{request_hash}`
- **Response:** `SignedCommitment`
- **Body:** `None`

- **Description**

    When supplied with a valid `request_hash`, this endpoint responds with the `SignedCommitment` object containing the same `SignedCommitment.Commitment.request_hash`. 

---

### **getSlots**

- **Method:** `GET /commitments/v0/gateway/slots`
- **Response:** `SlotInfoResponse`
- **Body:** `None`

- **Description**

    When called, the Gateway returns a `SlotInfo` for each upcoming L1 slot in the current or upcoming epoch. Each `SlotInfo` contains a list of `Offering` objects which specify the types of commitments they offer for a given chain, e.g., inclusion preconfs for the L1. 
    
    It should be noted that this endpoint does not provide guarantees that the Gateway is actually capable of providing these. For example, for proposer commitments that require delegations, the user should also consult the Constraints API to verify if the Gateway received delegations for the slot in question.  
---

### **getFeeInfo**

- **Method:** `POST /commitments/v0/gateway/fee`
- **Response:** `FeeInfo`
- **Headers:**
    - `Content-Type: application/json`
- **Body:** JSON object of type `CommitmentRequest`

- **Description**

    Since each proposer commitment protocol may have differing pricing mechanisms, i.e., per-request or subscription based, this endpoint is intentionally left generic. Users submit a `CommitmentRequest` and receive a `FeeInfo` object containing opaque `payload` bytes and a `commitment_type` to decode the `payload` into protocol-specific pricing information.

---

# Appendix
### Example - L1 Inclusion Preconf