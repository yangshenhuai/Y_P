---
title: "在浏览器中使用web3.js转ERC20 Token"
date: 2018-08-18T00:00:47+08:00
draft: false
tags: ["blockchain"]
author: "Peter Yang"
---
在2018年八月，用以下代码可以直接在浏览器中成功转ERC20的token

	<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/ethereum/web3.js@1.0.0-beta.34/dist/web3.min.js"></script>
	<script src="https://cdn.rawgit.com/ethereumjs/browser-builds/2fb69a714afe092b06645286f14b94f41e5c062c/dist/ethereumjs-tx.js"></script>	

	
	const web3 = new Web3(new Web3.providers.HttpProvider("https://ropsten.infura.io/v3/")) //使用infra
	var contractAddress = ""; //ERC20 token的地址
	var myAddress = ""; //from Account
	
	var abiArray = [{
	    "constant": true,
	    "inputs": [],
	    "name": "name",
	    "outputs": [{
	        "name": "",
	        "type": "string"
	    }],
	    "payable": false,
	    "stateMutability": "view",
	    "type": "function"
	}, {
	    "constant": false,
	    "inputs": [{
	        "name": "_spender",
	        "type": "address"
	    }, {
	        "name": "_value",
	        "type": "uint256"
	    }],
	    "name": "approve",
	    "outputs": [{
	        "name": "",
	        "type": "bool"
	    }],
	    "payable": false,
	    "stateMutability": "nonpayable",
	    "type": "function"
	}, {
	    "constant": true,
	    "inputs": [],
	    "name": "totalSupply",
	    "outputs": [{
	        "name": "",
	        "type": "uint256"
	    }],
	    "payable": false,
	    "stateMutability": "view",
	    "type": "function"
	}, {
	    "constant": false,
	    "inputs": [{
	        "name": "_from",
	        "type": "address"
	    }, {
	        "name": "_to",
	        "type": "address"
	    }, {
	        "name": "_value",
	        "type": "uint256"
	    }],
	    "name": "transferFrom",
	    "outputs": [{
	        "name": "",
	        "type": "bool"
	    }],
	    "payable": false,
	    "stateMutability": "nonpayable",
	    "type": "function"
	}, {
	    "constant": true,
	    "inputs": [],
	    "name": "INITIAL_SUPPLY",
	    "outputs": [{
	        "name": "",
	        "type": "uint256"
	    }],
	    "payable": false,
	    "stateMutability": "view",
	    "type": "function"
	}, {
	    "constant": true,
	    "inputs": [],
	    "name": "decimals",
	    "outputs": [{
	        "name": "",
	        "type": "uint8"
	    }],
	    "payable": false,
	    "stateMutability": "view",
	    "type": "function"
	}, {
	    "constant": false,
	    "inputs": [{
	        "name": "_spender",
	        "type": "address"
	    }, {
	        "name": "_subtractedValue",
	        "type": "uint256"
	    }],
	    "name": "decreaseApproval",
	    "outputs": [{
	        "name": "",
	        "type": "bool"
	    }],
	    "payable": false,
	    "stateMutability": "nonpayable",
	    "type": "function"
	}, {
	    "constant": true,
	    "inputs": [{
	        "name": "_owner",
	        "type": "address"
	    }],
	    "name": "balanceOf",
	    "outputs": [{
	        "name": "balance",
	        "type": "uint256"
	    }],
	    "payable": false,
	    "stateMutability": "view",
	    "type": "function"
	}, {
	    "constant": true,
	    "inputs": [],
	    "name": "symbol",
	    "outputs": [{
	        "name": "",
	        "type": "string"
	    }],
	    "payable": false,
	    "stateMutability": "view",
	    "type": "function"
	}, {
	    "constant": false,
	    "inputs": [{
	        "name": "_to",
	        "type": "address"
	    }, {
	        "name": "_value",
	        "type": "uint256"
	    }],
	    "name": "transfer",
	    "outputs": [{
	        "name": "",
	        "type": "bool"
	    }],
	    "payable": false,
	    "stateMutability": "nonpayable",
	    "type": "function"
	}, {
	    "constant": false,
	    "inputs": [{
	        "name": "_spender",
	        "type": "address"
	    }, {
	        "name": "_addedValue",
	        "type": "uint256"
	    }],
	    "name": "increaseApproval",
	    "outputs": [{
	        "name": "",
	        "type": "bool"
	    }],
	    "payable": false,
	    "stateMutability": "nonpayable",
	    "type": "function"
	}, {
	    "constant": true,
	    "inputs": [{
	        "name": "_owner",
	        "type": "address"
	    }, {
	        "name": "_spender",
	        "type": "address"
	    }],
	    "name": "allowance",
	    "outputs": [{
	        "name": "",
	        "type": "uint256"
	    }],
	    "payable": false,
	    "stateMutability": "view",
	    "type": "function"
	}, {
	    "inputs": [],
	    "payable": false,
	    "stateMutability": "nonpayable",
	    "type": "constructor"
	}, {
	    "anonymous": false,
	    "inputs": [{
	        "indexed": true,
	        "name": "owner",
	        "type": "address"
	    }, {
	        "indexed": true,
	        "name": "spender",
	        "type": "address"
	    }, {
	        "indexed": false,
	        "name": "value",
	        "type": "uint256"
	    }],
	    "name": "Approval",
	    "type": "event"
	}, {
	    "anonymous": false,
	    "inputs": [{
	        "indexed": true,
	        "name": "from",
	        "type": "address"
	    }, {
	        "indexed": true,
	        "name": "to",
	        "type": "address"
	    }, {
	        "indexed": false,
	        "name": "value",
	        "type": "uint256"
	    }],
	    "name": "Transfer",
	    "type": "event"
	}]


	function financialMfil(numMfil) {
    	return Number.parseFloat(numMfil / 1e3).toFixed(3);
	}

	function transferToken(account){
	    console.log(`web3 version: ${web3.version}`)
	    const trasnfer = async () => {
	    // This code was written and tested using web3 version 1.0.0-beta.29
	    // Who holds the token now?
	    
	    // Who are we trying to send this token to?
	    var destAddress = account;
	    // MineFIL Token (MFIL) is divisible to 3 decimal places, 1 = 0.001 of MFIL
	    var transferAmount = 1;
	    // Determine the nonce
	    var count = await web3.eth.getTransactionCount(myAddress);
	    console.log(`num transactions so far: ${count}`);
	    // MineFILToekn contract ABI Array
	    
	    // The address of the contract which created MFIL
	    var contract = new web3.eth.Contract(abiArray, contractAddress, {
	        from: myAddress
	    });
	    // How many tokens do I have before sending?
	    var balance = await contract.methods.balanceOf(myAddress).call();
	    console.log(`Balance before send: ${financialMfil(balance)} MFIL\n------------------------`);
	    // I chose gas price and gas limit based on what ethereum wallet was recommending for a similar transaction. You may need to change the gas price!
	    // Use Gwei for the unit of gas price
	    var gasPriceGwei = 4;
	    var gasLimit = 4000000;
	    // Chain ID of Ropsten Test Net is 3, replace it to 1 for Main Net
	    var chainId = 3;
	    console.info('the myAddress is ' + myAddress)
	    console.info('the contractAddress is ' + contractAddress)
	    console.info('the destAddress is ' + destAddress)
	    var rawTransaction = {
	        "from": myAddress,
	        "nonce": "0x" + count.toString(16),
	        "gasPrice": web3.utils.toHex(gasPriceGwei * 1e9),
	        "gasLimit": web3.utils.toHex(gasLimit),
	        "to": contractAddress,
	        "value": "0x0",
	        "data": contract.methods.transfer(destAddress, transferAmount).encodeABI(),
	        "chainId": chainId
	    };
	    console.log(`Raw of Transaction: \n${JSON.stringify(rawTransaction, null, '\t')}\n------------------------`);
	    var privKey = new EthJS.Buffer.Buffer("", 'hex'); //要加上转账的那个private key
	    var tx = new EthJS.Tx(rawTransaction);
	    tx.sign(privKey);
	    var serializedTx = tx.serialize();
	    // Comment out these four lines if you don't really want to send the TX right now
	    console.log(`Attempting to send signed tx:  ${serializedTx.toString('hex')}\n------------------------`);
	    var receipt = await web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'));
	    // The receipt info of transaction, Uncomment for debug
	    console.log(`Receipt info: \n${JSON.stringify(receipt, null, '\t')}\n------------------------`);
	    // The balance may not be updated yet, but let's check
	    balance = await contract.methods.balanceOf(myAddress).call();
	    console.log(`Balance after send: ${financialMfil(balance)} MFIL`);
	}
    	trasnfer()
	}