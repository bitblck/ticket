<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <script src="https://cdn.jsdelivr.net/gh/ethereum/web3.js@1.0.0-beta.34/dist/web3.min.js"></script>
</head>
<body>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
    <link rel="stylesheet" href="https://www.w3schools.com/lib/w3-colors-flat.css">

    <style>
        .center {
          margin: auto;
          width: 60%;
          padding: 10px;
        }
        </style>

<div class="w3-display-container center">

<div class="w3-card w3-red">
    <h2>Event Ticketing : using ERC721</h2>
</div>

<div class="w3-card">
    <p><Strong>Notes:</Strong>
    
    Requires the Use MetaMask - ticket seller/buyer address used is selected at MetaMask.
    Currently using <strong>Kovan</strong> TestNet</p>
 
  </div>
  <br> 
  <div class="w3-card w3-flat-green-sea">
    <table>
    <tr><td><strong>ERC20 Token Address</strong></td>
    <td><input type="text" id="token-address" size="50" value="0xe394fa049559db1166184Ca305F5b6C18F0D20F5"></input></td></tr>
    
    <tr><td><strong>Event Contract Address</strong></td>
    <td><input type="text" id="ticket-address" size="50" value="0x8bA6dEf379495Dad124C2fcaeBfC3D887b52D62A"></input></td></tr>
    <tr><td><strong>Current EOA Address</strong></td>
      <td><input type="text" id="eoa-address" size="50" value="unknown" readonly></input> Selected at MetaMask</td></tr>
    </table>
    <!--input type="number" id="qty" size="5"></input-->
  </div>
  
  <br>
  Ticket price and quantity are both fixed at one (1).
  <div class="w3-card w3-flat-green-sea">
    <table>
     <tr><td>
    <div><button id="approveTicketBtn" onclick="approveTicket()" style="width:250px">Approve Funds for deduction</button></div>
    </td><td>Prior approval required to allow Contracts to transfer Funds on your behalf</td></tr>
     <tr><td>
    <div><button id="buyTicketBtn" onclick="buyTicket()" style="width:250px">Buy Ticket from Organizer</button></div>
  </td><td>Transfer funds and Ticket</td></tr>
  <tr><td>
    <div><button id="buyTicketResellerBtn" onclick="buyTicketReseller()" style="width:250px">Buy Ticket from Reseller</button>
    </td><td><strong>Reseller Address</strong> <input type="text" id="seller-address" size="50"></input></div></td>
  </td></tr>
  <tr><td>
    <div><button id="sellTicketBtn" onclick="ticketToSell()" style="width:250px">Ticket Available for Sale</button></div>
  </td><td>Ticket is Marked for Sale</td></tr>
  <tr><td>
    <div><button id="balTicketBtn" onclick="balTicket()"  style="width:250px">Check Ticket Balance</button></div>
  </td></tr>
  </table>
  <br>
  </div>
  
  <div class="w3-card w3-cyan">
    <h3>Result</h3>
    <p><span id="approve-result"></span> </p>
    <p><span id="buy-result"></span></p>
    <p><span id="balance-result"></span></p>
    <p><span id="sell-result"></span></p>
    <br>
    <br>
  </div>
  
  <div class="w3-card w3-teal">
    <h3>Instructions on how to Test</h3>
  </div>
  
  <div class="w3-card">
    <OL><LI>First get some Ether from the network. Google for faucet.</LI>
    <LI>Then get some ERC20 Tokens for the DApp at the ERC20 Token Address. Send a ether transaction to the ERC20 Token Address (using Metamask). 
      It will act as a faucet.<br>
     <li> Add both Token Addresses in Metamask so that you can see how much tokens you have.</li>
    </LI>
      <LI>Create 2 externally owned accounts (EOA) on Metamask</LI>
    <p>[Note: EOA = Externally Owned Accounts, i.e. non-contract accounts]</p>
  <li>Buy ticket with EOA-1 from organizer. 
    <ol><li>Switch Metamask to EOA-1.</li>
        <li>Click: <strong>Approve Funds for Deduction.</strong></li>
        <li>Click:<strong>Buy Ticket from Organizer</strong></li>
    <li>Click: <strong>Check Ticket Balance</strong> (may take a while)</li>
    </ol>
    </li>
    <LI>Buy ticket from Reseller. 
    <ol>
    <li>Switch to EOA-1 (which have just bought a ticket)</li>
    <li>Click: <strong>Ticket Available For Sale</strong></li>
    <li>Switch to EOA-2.</li>
    <li>Click: <strong>Approve Funds for Deduction.</strong></li>
    <li>Add EOA-1 address in text field. Click: <strong>Buy Ticket from Reseller.</strong></li>
    <li>Click: <strong>Check Ticket Balance</strong> (may take a while)</li>
  </ol>
  </LI>
  </OL>
  </div>
  <br>

</div>

<script>

    var web3;
    var tokenContract;
	  var ticketContract;
    var ticketAddress;
    var tokenAddress;
    var account;
    var decimal;

	function getEventTicketContract(TicketAddress){
		let minABI=[
    { "constant": false, "inputs": [ { "name": "to", "type": "address" }, { "name": "tokenId", "type": "uint256" } ], "name": "approve", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "buyer", "type": "address" } ], "name": "buyTicket", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "seller", "type": "address" }, { "name": "buyer", "type": "address" } ], "name": "buyTicketFromReseller", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "seller", "type": "address" } ], "name": "markForSale", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "from", "type": "address" }, { "name": "to", "type": "address" }, { "name": "tokenId", "type": "uint256" } ], "name": "safeTransferFrom", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "from", "type": "address" }, { "name": "to", "type": "address" }, { "name": "tokenId", "type": "uint256" }, { "name": "_data", "type": "bytes" } ], "name": "safeTransferFrom", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "to", "type": "address" }, { "name": "approved", "type": "bool" } ], "name": "setApprovalForAll", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "anonymous": false, "inputs": [ { "indexed": true, "name": "from", "type": "address" }, { "indexed": true, "name": "to", "type": "address" }, { "indexed": true, "name": "tokenId", "type": "uint256" } ], "name": "Transfer", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "name": "owner", "type": "address" }, { "indexed": true, "name": "approved", "type": "address" }, { "indexed": true, "name": "tokenId", "type": "uint256" } ], "name": "Approval", "type": "event" }, { "anonymous": false, "inputs": [ { "indexed": true, "name": "owner", "type": "address" }, { "indexed": true, "name": "operator", "type": "address" }, { "indexed": false, "name": "approved", "type": "bool" } ], "name": "ApprovalForAll", "type": "event" }, { "constant": false, "inputs": [ { "name": "buyer", "type": "address" } ], "name": "transferERC20Token", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": false, "inputs": [ { "name": "from", "type": "address" }, { "name": "to", "type": "address" }, { "name": "tokenId", "type": "uint256" } ], "name": "transferFrom", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "inputs": [ { "name": "name", "type": "string" }, { "name": "symbol", "type": "string" }, { "name": "maxSupply", "type": "uint256" }, { "name": "ticketPrice", "type": "uint256" }, { "name": "startDate", "type": "uint256" }, { "name": "endDate", "type": "uint256" }, { "name": "ERC20Address", "type": "address" } ], "payable": false, "stateMutability": "nonpayable", "type": "constructor" }, { "constant": false, "inputs": [ { "name": "seller", "type": "address" } ], "name": "unmarkForSale", "outputs": [], "payable": false, "stateMutability": "nonpayable", "type": "function" }, { "constant": true, "inputs": [ { "name": "owner", "type": "address" } ], "name": "balanceOf", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "baseURI", "outputs": [ { "name": "", "type": "string" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "tokenId", "type": "uint256" } ], "name": "getApproved", "outputs": [ { "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "seller", "type": "address" } ], "name": "getTokenIdForSale", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "owner", "type": "address" }, { "name": "operator", "type": "address" } ], "name": "isApprovedForAll", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "seller", "type": "address" } ], "name": "isMarkForSale", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "name", "outputs": [ { "name": "", "type": "string" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "tokenId", "type": "uint256" } ], "name": "ownerOf", "outputs": [ { "name": "", "type": "address" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "interfaceId", "type": "bytes4" } ], "name": "supportsInterface", "outputs": [ { "name": "", "type": "bool" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "symbol", "outputs": [ { "name": "", "type": "string" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "buyer", "type": "address" } ], "name": "testEnoughFunds", "outputs": [ { "name": "ret", "type": "bool" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "index", "type": "uint256" } ], "name": "tokenByIndex", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "owner", "type": "address" }, { "name": "index", "type": "uint256" } ], "name": "tokenOfOwnerByIndex", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [ { "name": "tokenId", "type": "uint256" } ], "name": "tokenURI", "outputs": [ { "name": "", "type": "string" } ], "payable": false, "stateMutability": "view", "type": "function" }, { "constant": true, "inputs": [], "name": "totalSupply", "outputs": [ { "name": "", "type": "uint256" } ], "payable": false, "stateMutability": "view", "type": "function" } ]
		;
		return new web3.eth.Contract(minABI, TicketAddress);
    }
	
    function getERC20TokenContract(tokenAddress) {
      let minABI = [
   {"constant":false,"inputs":[{"name":"account","type":"address"}],"name":"addPauser","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},
	 {"constant":false,"inputs":[{"name":"spender","type":"address"},{"name":"value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
	 {"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"burn","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
	 {"constant":false,"inputs":[{"name":"spender","type":"address"},{"name":"subtractedValue","type":"uint256"}],"name":"decreaseAllowance","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
	 {"constant":false,"inputs":[{"name":"spender","type":"address"},{"name":"addedValue","type":"uint256"}],"name":"increaseAllowance","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
	 {"constant":false,"inputs":[],"name":"pause","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"anonymous":false,"inputs":[{"indexed":false,"name":"account","type":"address"}],"name":"Paused","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"account","type":"address"}],"name":"PauserAdded","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"account","type":"address"}],"name":"PauserRemoved","type":"event"},
	 {"constant":false,"inputs":[],"name":"renounceOwnership","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"renouncePauser","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"to","type":"address"},{"name":"value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
	 {"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Transfer","type":"event"},
	 {"anonymous":false,"inputs":[{"indexed":true,"name":"owner","type":"address"},{"indexed":true,"name":"spender","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Approval","type":"event"},
	 {"anonymous":false,"inputs":[{"indexed":true,"name":"previousOwner","type":"address"},{"indexed":true,"name":"newOwner","type":"address"}],"name":"OwnershipTransferred","type":"event"},
	 {"constant":false,"inputs":[{"name":"from","type":"address"},{"name":"to","type":"address"},{"name":"value","type":"uint256"}],"name":"transferFrom","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},
	 {"constant":false,"inputs":[{"name":"newOwner","type":"address"}],"name":"transferOwnership","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},
	 {"constant":false,"inputs":[],"name":"unpause","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},
	 {"anonymous":false,"inputs":[{"indexed":false,"name":"account","type":"address"}],"name":"Unpaused","type":"event"},
	 {"constant":true,"inputs":[{"name":"owner","type":"address"},{"name":"spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[{"name":"account","type":"address"}],"name":"balanceOf","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[],"name":"isOwner","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[{"name":"account","type":"address"}],"name":"isPauser","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[],"name":"paused","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},
	 {"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}
      ];
      return new web3.eth.Contract(minABI, tokenAddress);
    }
    function getEventTicketContractAddress() {
      ticketAddress = document.getElementById('ticket-address').value;
      if(web3.utils.isAddress(ticketAddress)){
        if(ticketAddress != "") {
          ticketContract = getEventTicketContract(ticketAddress);
          console.log("got contractAddress "+ticketAddress)
        }
      }
      else {
        alert("Contract Address failed checksum!");
      }
    }

    function getTokenContractAddress() {
      tokenAddress = document.getElementById('token-address').value;
      if(web3.utils.isAddress(tokenAddress)){
        if(tokenAddress != "") {
          tokenContract = getERC20TokenContract(tokenAddress);
          console.log("got tokenAddress "+tokenAddress)
        }
      }
      else {
        alert("Token Address failed checksum!");
      }
    }
    function approveTransfer(buyer, value){ //value must equal ticket price
	  //buyer approve of ticket contract to transfer fees on her behalf
      tokenContract.methods.approve(ticketAddress,value).send({from: buyer})
      .on('transactionHash', (txHash) => {
        document.getElementById('approve-result').innerText = "Approve Success - Transaction Hash = " + txHash;
        document.getElementById('balance-result').innerText="";
      })
      .on('error', (error, receipt) => {
        document.getElementById('approve-result').innerText = "Approve Error: "+ buyer+" "+ error;
        document.getElementById('balance-result').innerText="";
        //log error address here
        console.log('Error : '+buyer);
      });
      console.log("buyer="+buyer);
    }

    function transferToken(buyer) {
	  //ticket contract transfer funds from buyer account which was prior approved
      ticketContract.methods.buyTicket(buyer).send({from: buyer})
      .on('transactionHash', (txHash) => {
        document.getElementById('buy-result').innerText = "Transfer Success - Transaction Hash = " + txHash;
        document.getElementById('balance-result').innerText="";
      })
      .on('error', (error, receipt) => {
        document.getElementById('buy-result').innerText = "Transfer Error: "+ buyer+" "+ error;
        document.getElementById('balance-result').innerText="";
        //log error address here
        console.log('Error : '+buyer);
      });
    }
    function approveTicket(){
      approveTransfer(account,1);
    }
    function buyTicket() {
      //transfer to EventTicket Contract
      transferToken(account); //should get ticket price first
    }

    function buyTicketReseller(){
      seller = document.getElementById('seller-address').value;
      console.log("seller: "+seller);
	  //ticket contract transfer funds from buyer account (prior approval required) to reseller account
    ticketContract.methods.buyTicketFromReseller(seller, account).send({from: account})
      .on('transactionHash', (txHash) => {
        document.getElementById('buy-result').innerText = "Transfer Success - Transaction Hash = " + txHash;
        document.getElementById('balance-result').innerText="";
      })
      .on('error', (error, receipt) => {
        document.getElementById('buy-result').innerText = "Transfer Error: "+ account+" "+ error;
        //log error address here
        document.getElementById('balance-result').innerText="";
        console.log('Error : '+account);
      });
    }

    function balTicket(){
      getBalanceOf(account);
    }
    function getBalanceOf(buyer){
      getEventTicketContractAddress();
      ticketContract.methods.symbol().call().then((result)=>{
          //document.getElementById('balance-result').innerText="Ticket Symbol="+result;
          sym=result;
          console.log(result);
      });
      ticketContract.methods.balanceOf(buyer).call().then((result)=>{
          document.getElementById('balance-result').innerText="Ticket "+sym+" Balance="+result;
          console.log(result);
          document.getElementById('buy-result').innerText = "";
          document.getElementById('sell-result').innerText="";
          document.getElementById('approve-result').innerText ="";
      });
    }
    function getERC20TokenBalance(walletAddress, callback) {

        tokenContract.methods.balanceOf(walletAddress).call().then((result)=>{
          var res = web3.utils.toBN(result);
          res = res / (10 ** decimal);
          document.getElementById('balance-result').innerText=res;
          console.log(res)
      });
    }
    function ticketToSell(){
      var res=0;
      ticketContract.methods.markForSale(account).send({from: account}).then((result)=>{
        console.log(result);
        ticketContract.methods.getTokenIdForSale(account).call().then((result)=>{
          document.getElementById('sell-result').innerText=result;
          res=result; //should tokenId if greater than 0
          console.log('Sell='+result);
          if(res>0){
            ticketContract.methods.approve(ticketAddress,res).send({from: account}).then((result)=>{
              document.getElementById('sell-result').innerText+=" approved";
              console.log('Sell='+result);
              });
          }
        });
      });
    }
    function getBalance(){
      var tokenAddress = document.getElementById('token-address').value;
      var walletAddress = document.getElementById('wallet-address').value;
      let decimals = web3.utils.toBN(document.getElementById('decimals').value);
      var amount = web3.utils.toBN(document.getElementById('amount').value);
      
      var sendValue = amount.mul(web3.utils.toBN(10).pow(decimals));
      console.log("decimals "+decimals.toString());
      getTokenContractAddress();
      getERC20TokenBalance(walletAddress);
    }
    function getERC20TokenDecimals(callback) {
      tokenContract.methods.decimals().call((error, decimals) => {
        callback(decimals);
        decimal=decimals;
      });
    }
    window.addEventListener('load', async () => {
    // Modern dapp browsers...
    if (window.ethereum) {
      window.web3 = new Web3(ethereum);
      try {
        // Request account access if needed
        await ethereum.enable();
        console.log(web3.version);
        getAccounts();
        // send_something();
      } catch (error) {
        console.log(error);
        // User denied account access...
      }
    }
    // Legacy dapp browsers...
    else if (window.web3) {
      window.web3 = new Web3(web3.currentProvider);
      console.log(web3.version);
      getaccounts();
    }
    // Non-dapp browsers...
    else {
      console.log('Non-Ethereum browser detected. You should consider trying MetaMask!');
  }
});
function setAccount(){
    document.getElementById('eoa-address').value = account;
}

function getAccounts(){
  var accountInterval = setInterval(function() {
        web3.eth.getAccounts((error, address) => {
          if (address[0] !== account) {
            account = address[0];
            console.log("Account: "+account);
            setAccount();
          }
        });

      }, 300);
		  getTokenContractAddress();
		  getEventTicketContractAddress();
      
}


  </script>
 
</body>
</html>
