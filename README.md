# azure-network-inflate

This project, **azure-network-inflate**, is designed to simplify and automate the process of managing Azure network configurations. It provides tools and scripts to streamline network setup, scaling, and optimization.

## Features

- Automates Azure network resource creation.
- Supports scaling and optimization of network configurations.
- Provides detailed logging and error handling.
- Easy-to-use and customizable scripts.

## Prerequisites

Before using this project, ensure you have the following:

- An active Azure subscription.
- Service principals for both frontend and backend authentication.
- Create .env file at root level
- A `.env` configuration file with the following variables:
    ```bash
    AZURE_FRONTEND_APP_CLIENT_ID=<Frontend App Client ID from Azure AD>
    AZURE_TENANT_ID=<Azure Active Directory Tenant ID>
    AZURE_BACKEND_APP_CLIENT_ID=<Backend App Client ID from Azure AD>
    AZURE_BACKEND_APP_SECRET=<Backend App Client Secret from Azure AD>
    AZURE_SUBSCRIPTION_ID=<Azure Subscription ID>
    AZURE_BACKENDAUTH_CLIENT_ID=<Backend Authentication Client ID>
    ```
- Python 3.8 or higher (if you plan to use the provided scripts).


## Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/GaneshGoud5688/azure-network-inflate.git
    cd azure-network-inflate
    ```

2. Create and activate a virtual environment (optional but recommended):
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    ```

3. Install dependencies:
    ```bash
    pip install -r requirements.txt
    ```

## Usage

1. Configure your Azure credentials and subscription.
2. Run the main script or command:
    ```bash
    uvicorn main:app --reload
    ```

# Azure VNet Provisioner API Set

This API set provides endpoints for managing Azure Virtual Networks (VNets) and their subnets. It is secured using Azure Active Directory authentication.

## Endpoints

### 1. Root Endpoint
**`GET /`**

- **Description**: Returns a message indicating that the API is operational.
- **Authentication**: Required.

---

### 2. Create or Update Virtual Network
**`POST /vnets`**
- **Description**: Creates or updates a virtual network based on the provided configuration.

| Parameter       | Type       | Required | Description                          | Example Data                          |
|-----------------|------------|----------|--------------------------------------|---------------------------------------|
| `resource_group` | `string`   | Yes      | Name of the resource group.          | `"demo-rg"`                           |
| `vnet_name`      | `string`   | Yes      | Name of the virtual network.         | `"myVNet"`                            |
| `location`       | `string`   | Yes      | Azure region for the VNet.           | `"eastus"`                            |
| `address_prefix` | `string`   | Yes      | Address space for the VNet.          | `"10.0.0.0/16"`                       |
| `subnets`        | `list`     | No       | List of subnets for the VNet.        | `[{"name": "subnet1", "address_prefix": "10.0.1.0/24"}]` |

## Example Usage
```json
{
    "resource_group": "demo-rg",
    "vnet_name": "myVNet",
    "location": "eastus",
    "address_prefix": "10.0.0.0/16",
    "subnets": [
        {
            "name": "subnet1",
            "address_prefix": "10.0.1.0/24"
        },
        {
            "name": "subnet2",
            "address_prefix": "10.0.2.0/24"
        },
        {
            "name": "subnet3",
            "address_prefix": "10.0.3.0/24"
        }
    ]
}
```
---

### 3. Retrieve VNet Data
**`GET /vnets`**

- **Description**: Retrieves details about a specific VNet or all VNets.
- **Authentication**: Required.

| Parameter   | Type       | Required | Description                          | Example Data |
|-------------|------------|----------|--------------------------------------|--------------|
| `vnet_name`  | `string`   | No       | Name of the virtual network to fetch. | `"my-vnet"`  |

---

### 4. Delete a Subnet
**`DELETE /vnets/subnet`**

- **Description**: Deletes a subnet from a specified VNet in a given resource group.
- **Authentication**: Required.

| Parameter       | Type       | Required | Description                          | Example Data                          |
|-----------------|------------|----------|--------------------------------------|---------------------------------------|
| `resource_group` | `string`   | Yes      | Name of the resource group.          | `"my-resource-group"`                 |
| `vnet_name`      | `string`   | Yes      | Name of the virtual network.         | `"my-vnet"`                           |
| `subnet_name`    | `string`   | Yes      | Name of the subnet to delete.        | `"subnet1"`                           |

## Example Usage
```bash
{
"resource_group": "demo-rg",
"vnet_name": "myVNet",
"subnet_name": "subnet2"
}
```
---

### 5. Delete a Virtual Network
**`DELETE /vnets`**

- **Description**: Deletes a specified virtual network.
- **Authentication**: Required.

| Parameter       | Type       | Required | Description                          | Example Data                          |
|-----------------|------------|----------|--------------------------------------|---------------------------------------|
| `resource_group` | `string`   | Yes      | Name of the resource group.          | `"my-resource-group"`                 |
| `vnet_name`      | `string`   | Yes      | Name of the virtual network.         | `"my-vnet"`                           |


## Example Usage
```bash
{
"resource_group": "demo-rg",
"vnet_name": "myVNet"
}
```
---

## Authentication
- All endpoints are secured using Azure Active Directory authentication. Ensure you provide a valid token when accessing the API.
---

## Why FastApi
- **High Performance** – As fast as Node.js and Go (thanks to Starlette + Pydantic).
- **Easy to Use** – Minimal code, automatic docs (Swagger, ReDoc).
- **Type-Safe** – Uses Python type hints for validation and autocompletion.
- **Async Support** – Built-in support for async/await for better concurrency.
- **Dependency Injection** – Clean, testable, and scalable architecture.
- **Security** – Easy OAuth2, JWT, and API key integration.
- **Auto Validation** – Request data is validated automatically using Pydantic.


---
## Here are comprehensive test scenarios for your Azure VNET API requirements, covering functionality, security, and edge cases:

### 1. Authentication
- Attempt to access all endpoints without authentication token  
- Expected: 401 Unauthorized for all requests

### 2. VNET Creation Tests
2.1 *Valid VNET Creation*  
- POST /vnets with valid payload (name, address space, subnets)  
- Expected: 201 Created, returns VNET details with provisioning state  

2.2 *Missing Required Fields*  
- POST /vnets missing 'name' or 'address_space'  
- Expected: 400 Bad Request with error details  

### 3. Subnet Tests
3.1 *Multiple Valid Subnets*  
- Create VNET with 3 properly configured subnets  
- Expected: All subnets created with correct CIDR ranges  

3.2 *Existing Valid Subnet with no change*  
- Update VNET with 3 properly configured existing subnets with no change  
- Expected: All subnets as it is no change  

3.3 *Existing Valid Subnet change in CIDR*  
- Update VNET with subnet to be updated which is properly configured existing subnets.
- Expected: Update in respective subnet

### 4. Data Retrieval Tests
4.1 *Get All VNETs*  
- GET /vnets after creating multiple VNETs  
- Expected: 200 OK with complete list  

4.2 *Get Specific VNET*  
- GET /vnets/{vnet_name}  
- Expected: 200 OK with full details including subnets  

### 8. Negative Tests
8.1 *Malformed JSON*  
- POST /vnets with invalid JSON body  
- Expected: 400 Bad Request  

8.2 *Wrong HTTP Methods*  
- PUT/PATCH/DELETE on /vnets if not implemented  
- Expected: 405 Method Not Allowed  
