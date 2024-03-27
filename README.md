# DecentralizedHealth-Web3-Based-Healthcare-Data-Management
DecentralizedHealth uses blockchain technology to securely store and manage patient health records, ensuring data privacy and enabling patients to control access to their information.
from web3 import Web3
from solcx import compile_source
from web3.middleware import geth_poa_middleware

# Connect to Ethereum node (Example with Infura)
infura_url = 'https://mainnet.infura.io/v3/YOUR_INFURA_PROJECT_ID'
web3 = Web3(Web3.HTTPProvider(infura_url))

# Add POA middleware for the Rinkeby testnet or other POA networks
web3.middleware_onion.inject(geth_poa_middleware, layer=0)

# Check connection
if web3.isConnected():
    print("Connected to Ethereum network")
else:
    print("Failed to connect to Ethereum network")

# Smart Contract Source Code (Solidity)
contract_source_code = '''
pragma solidity ^0.8.0;

contract DecentralizedHealth {
    struct HealthRecord {
        uint id;
        string data;
    }

    mapping(uint => HealthRecord) public healthRecords;
    uint public nextId;

    function createHealthRecord(string memory data) public {
        healthRecords[nextId] = HealthRecord(nextId, data);
        nextId++;
    }

    function getHealthRecord(uint id) public view returns (string memory) {
        return healthRecords[id].data;
    }
}
'''

# Compile the contract
compiled_sol = compile_source(contract_source_code, output_values=['abi', 'bin'])
contract_id, contract_interface = compiled_sol.popitem()
bytecode = contract_interface['bin']
abi = contract_interface['abi']

# Deploy the contract (Assuming you have an account set up)
account = web3.eth.account.privateKeyToAccount('YOUR_PRIVATE_KEY')
web3.eth.defaultAccount = account.address

# Construct the contract
DecentralizedHealth = web3.eth.contract(abi=abi, bytecode=bytecode)

# Submit the transaction that deploys the contract
tx_hash = DecentralizedHealth.constructor().transact()
tx_receipt = web3.eth.waitForTransactionReceipt(tx_hash)

# Contract is now deployed
contract_address = tx_receipt.contractAddress
print(f"Contract deployed at address: {contract_address}")

# Interacting with the contract
dh = web3.eth.contract(address=contract_address, abi=abi)

# Example: Adding a health record
tx_hash = dh.functions.createHealthRecord('Patient A Health Data').transact()
tx_receipt = web3.eth.waitForTransactionReceipt(tx_hash)
print("Added new health record.")

# Example: Retrieving a health record
health_record = dh.functions.getHealthRecord(0).call()
print(f"Health Record Data: {health_record}")
