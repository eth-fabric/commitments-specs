# Commitments API Specification

## Abstract

todo

## Motivation

todo


## API Scope

**In Scope**

todo

**Out of Scope**

todo

# Terminology

| Term           | Description                                                                                                                                    |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| Todo      | todo  |


# API Summary

| **Namespace** | **Method** | **Endpoint** | **Description** |
| --- | --- | --- | --- |
| `commitments`   | `POST` | [/commitments/v0/todo](./commitments-api.md#endpoint-commitmentsv0todo)        | ... |

# Commitments API Endpoints

### Endpoint: `/commitments/v0/todo`

Overview todo

- **Method:** `POST`
- **Response:** Empty
- **Headers:**
    - `Content-Type: application/json`
- **Body:** JSON object of type `SignedConstraints[]`

- **Schema**

    ```python
    # A signed "bundle" of constraints.
    class SomeClass(Container):
        pass
    ```

- **Description**

    todo