# Creating a Celo Payment Gateway-

## Introduction:

We are going to create a Celo payment gateway using React, TypeScript, and the Celo SDK in this tutorial. Users will be able to take payments in Celo Stablecoins through the payment gateway, which will provide a secure and transparent payment mechanism. The Celo ContractKit library will be used to communicate with the Celo blockchain, and the React framework will be used to create the user interface.

## Requirements:

You will need the following to complete this tutorial: 

- Knowledge of [TypeScript](https://www.typescriptlang.org/download) and [React](https://react-cn.github.io/react/downloads.html).
- Install [Node.js](https://nodejs.org/en/download) (v16 and above)
- Install [npm](https://www.npmjs.com/package/download)  
- Creation of Celo account for testing the payment gateway & depositing Celo stable coins.

## 1. Create a Celo Node:

First step is to set up a Celo node on your server to create a Celo payment gateway. To set it up we first, need to install the Celo Cli and this can be installed through the following code:

```
` npm install -g celocli `
```

After installing the Celo CLI, run the following command to start your Celo node:

```
`celocli init`
```

To configure your Celo node, according to the instructions. Your account address, password, and other information will be requested.

## 2. Create a smart contract:

The next step is creation of the Smart Contract is to control the payment gateway. Solidity, a programming language that is used for creating Smart Contracts on the Ethereum network.

For your project, create a new directory and then create a file called PaymentGateway.sol with the following code:

```solidity
// Set the Solidity compiler version to 0.8.0 or later
pragma solidity ^0.8.0;

// Import the Address library from the Celo ContractKit
import "@celo/contractkit/contracts/utils/Address.sol";

// Define the PaymentGateway contract
contract PaymentGateway {
    // Use the Address library for address payable
    using Address for address payable;

    // Declare a public owner variable of type address
    address public owner;

    // Declare a mapping to store balances for each address
    mapping(address => uint256) public balances;

    // Define the constructor function, which sets the contract owner to the address that deployed the contract
    constructor() {
        owner = msg.sender;
    }

    // Define the pay function, which receives and adds funds to the sender's balance
    function pay() public payable {
        balances[msg.sender] += msg.value;
    }

    // Define the withdraw function, which withdraws funds from the sender's balance and sends them to the sender's address
    function withdraw(uint256 amount) public {
        // Require that the sender has enough balance to withdraw the specified amount
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // Subtract the withdrawn amount from the sender's balance
        balances[msg.sender] -= amount;

        // Transfer the withdrawn amount to the sender's address
        payable(msg.sender).transfer(amount);
    }

    // Define the receive function, which is called when the contract receives funds without a function call
    receive() external payable {}

    // Define the fallback function, which is called when the contract receives a function call that does not exist
    fallback() external payable {}

    // Define events to track fund deposits and withdrawals
    event FundsDeposited(address indexed account, uint256 amount);
    event FundsWithdrawn(address indexed account, uint256 amount);
}

```

The pay feature of this Smart Contract updates the user's balance on the Celo blockchain and takes payments in Celo Stablecoins. Additionally, it offers a withdraw feature that enables customers to get their remaining amounts.

## 3. Deploy the smart contract:

Once the smart contract has been created, it must be deployed to the Celo network using the Celo CLI. In this step, the Smart Contract code is compiled, the ABI (Application Binary Interface) is produced, and the contract is deployed to the Celo blockchain.

Run the next command to construct the smart contract:

`celocli compile PaymentGateway.sol`

By doing this, the ABI and bytecode for the smart contract will be generated.

Run the following command to deploy the smart contract on the Celo blockchain.

`celocli deploy PaymentGateway --args '[]'`

## 4. Integrating the Celo SDK:

We must communicate with our Smart Contract from our React application now that it has been launched. The Celo SDK comes in help with this situation. We are given a collection of JavaScript libraries by the Celo SDK that we can use in our application to communicate with the Celo blockchain.

Installing the necessary prerequisites for the Celo SDK is the first step.

`npm install @celo/contractkit`

The ContractKit object will then be imported into our src/App.tsx file from the @celo/contractkit package:

`import { ContractKit } from '@celo/contractkit';`

The Celo network we want to connect to will then be initialized into an instance of the ContractKit object that we have just created. The user's Celo account address will also be kept in a new constant we'll name accounts:

```javascript
const contractKit = new ContractKit(web3Provider);

// Get the user's accounts
const accounts = await contractKit.web3.eth.getAccounts();
```

The user's address will be displayed in our user interface using the accounts constant.

The user will click the "Pay" button in our user interface, therefore we'll next construct a new function called sendPayment that will start a payment transaction when they do. The recipient's address and the money to be sent are the two inputs for the function.

```javascript
// Define an asynchronous function called sendPayment that takes in two parameters: recipientAddress and amount
const sendPayment = async (recipientAddress: string, amount: string) => {
  try {
    // Get the instance of the stable token contract from the contractKit object
    const stableToken = await contractKit.contracts.getStableToken();

    // Transfer the specified amount to the recipient address using the stableToken instance
    // The transfer function returns a transaction object that represents the transfer
    const txObject = await stableToken.transfer(recipientAddress, amount).send({ from: accounts[0] });
    
    // Wait for the transaction to be mined and get the receipt
    const txReceipt = await txObject.waitReceipt();

    // Log the transaction hash to the console
    console.log(`Transaction hash: ${txReceipt.transactionHash}`);
  } catch (error) {
    // If an error occurs during the execution of the function, log the error to the console
    console.log(error);
  }
}

```

Using the ContractKit object, we first obtain an instance of the StableToken contract in this function. Then, we start a payment transaction with the specified amount to the recipient's address using the transfer function. The from argument is set to the user's account address.

The transaction hash is then displayed in the console as we use the waitReceipt function to wait for the transaction to be mined.

## 5. Building the user interface:

We can begin creating our user interface now that our smart contract has been launched and our integration with the Celo SDK has been set up.

First, we'll make a brand-new file called PaymentForm.tsx in the src/components directory. We'll define a functional component that renders a form for the user to enter the recipient's address and other information in this file.

```react
// Import the React and useState modules from the 'react' package
import React, { useState } from 'react';

// Define a Props interface that describes the props for the PaymentForm component
interface Props {
  sendPayment: (recipientAddress: string, amount: string) => Promise<void>; // A function that sends a payment to a recipient address with a specified amount
}

// Define a functional component called PaymentForm that takes in the sendPayment function as a prop
const PaymentForm: React.FC<Props> = ({ sendPayment }) => {
  // Declare two state variables using the useState hook: recipientAddress and amount
  const [recipientAddress, setRecipientAddress] = useState('');
  const [amount, setAmount] = useState('');

  // Define a handleSubmit function that is called when the form is submitted
  const handleSubmit = async (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault(); // Prevent the default form submission behavior

    // Call the sendPayment function with the recipientAddress and amount state variables as arguments
    await sendPayment(recipientAddress, amount);

    // Clear the recipientAddress and amount state variables
    setRecipientAddress('');
    setAmount('');
  }

  // Render a form element with two input fields for recipientAddress and amount, and a submit button
  return (
    <form onSubmit={handleSubmit}>
      <label>
        Recipient Address:
        <input type="text" value={recipientAddress} onChange={(event) => setRecipientAddress(event.target.value)} />
      </label>
      <label>
        Amount:
        <input type="text" value={amount} onChange={(event) => setAmount(event.target.value)} />
      </label>
      <button type="submit">Send Payment</button>
    </form>
  );
}

```

## 6. Build a page that shows the logged-in merchant's payment history: 

Inside the src/pages directory, make a new file called PaymentHistory.tsx and add the following code:

```

import React, { useEffect, useState } from 'react';
import { Container, Table } from 'react-bootstrap';
import { useRecoilValue } from 'recoil';
import { userAddressState } from '../recoil/atoms';
import { Payment } from '../types';
import { getPaymentHistory } from '../utils/celo';
import { formatDate } from '../utils/date';

const PaymentHistory = () => {
  const userAddress = useRecoilValue(userAddressState);
  const [payments, setPayments] = useState<Payment[]>([]);

  useEffect(() => {
    const fetchPaymentHistory = async () => {
      const paymentHistory = await getPaymentHistory(userAddress);
      setPayments(paymentHistory);
    };

    if (userAddress) {
      fetchPaymentHistory();
    }
  }, [userAddress]);

  return (
    <Container className="my-5">
      <h2>Payment History</h2>
      <Table striped bordered hover>
        <thead>
          <tr>
            <th>Date</th>
            <th>Transaction Hash</th>
            <th>Amount</th>
          </tr>
        </thead>
        <tbody>
          {payments.map((payment) => (
            <tr key={payment.txHash}>
              <td>{formatDate(payment.date)}</td>
              <td>{payment.txHash}</td>
              <td>{`${payment.amount} ${payment.currency}`}</td>
            </tr>
          ))}
        </tbody>
      </Table>
    </Container>
  );
};

export default PaymentHistory;

```

The user's address is obtained from the Recoil state using the useRecoilValue hook in this code. Then, using the getPaymentHistory function we earlier established, we are using the useEffect hook to retrieve the payment history. The payments state variable, which is shown as a table using the React Bootstrap Table component, contains information about past payments.

The payment amount and currency are rendered, and the payment date is formatted using the formatDate function.

## 7. Update Navigation:

The menu will now be updated to include links to the payment and payment history pages. Replace the present code in the NavBar.tsx file located in the src/components directory with the following:

```react
import React from 'react';
import { Container, Nav, Navbar } from 'react-bootstrap';
import { Link } from 'react-router-dom';

const NavBar = () => {
  return (
    <Navbar bg="light" expand="lg">
      <Container>
        <Navbar.Brand as={Link} to="/">
          Celo Payment Gateway
        </Navbar.Brand>
        <Navbar.Toggle aria-controls="basic-navbar-nav" />
        <Navbar.Collapse id="basic-navbar-nav">
          <Nav className="me-auto">
            <Nav.Link as={Link} to="/">
              Home
            </Nav.Link>
            <Nav.Link as={Link} to="/payment">
              Payment
            </Nav.Link>
            <Nav.Link as={Link} to="/history">
              Payment History
            </Nav.Link>
          </Nav>
        </Navbar.Collapse>
      </Container>
    </Navbar>
  );
};

export default NavBar;
```

## 8. Create the PaymentForm Component:

Let's now build a component that will render a money transaction request form. The user can enter the desired payment amount using this component, which will accept the item and price as props.

In the components directory, make a new file called PaymentForm.tsx, and add the following code:

```react
import React, { useState } from "react";
import { ContractKit } from "@celo/contractkit";
import { PaymentRequest } from "../types";

interface PaymentFormProps {
  item: string;
  price: number;
  requestPayment: (paymentRequest: PaymentRequest) => void;
}

export default function PaymentForm({
  item,
  price,
  requestPayment,
}: PaymentFormProps) {
  const [amount, setAmount] = useState("");

  const handleAmountChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setAmount(event.target.value);
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();

    if (amount === "") {
      alert("Please enter a valid amount.");
      return;
    }

    const paymentRequest: PaymentRequest = {
      item,
      price,
      amount: parseFloat(amount),
    };

    requestPayment(paymentRequest);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="item">Item:</label>
        <span id="item">{item}</span>
      </div>
      <div>
        <label htmlFor="price">Price:</label>
        <span id="price">{price} CELO</span>
      </div>
      <div>
        <label htmlFor="amount">Amount:</label>
        <input
          type="number"
          id="amount"
          min="0"
          step="0.0001"
          value={amount}
          onChange={handleAmountChange}
          required
        />
        <span>CELO</span>
      </div>
      <button type="submit">Pay</button>
    </form>
  );
}
```

The item name, price, and an input field for the user to enter the payment amount are displayed on a form that is created as part of this component. Additionally, we develop event handlers for when the form is submitted and the amount changes.

Prior to submitting the form, we verify that a valid amount was input. If so, we create a payment request object and send it as a prop to the requestPayment function.

## 9. Create the app component:

Let's design the primary app component, which will render the PaymentForm component and control the payment process last.In the src directory, make a new file called App.tsx and add the following code:

```react
import React, { useState } from "react";
import { ContractKit } from "@celo/contractkit";
import PaymentForm from "./components/PaymentForm";
import { PaymentRequest } from "./types";

const contractAddress = "<YOUR_SMART_CONTRACT_ADDRESS>";

export default function App() {
  const [isProcessingPayment, setIsProcessingPayment] = useState(false);

  const requestPayment = async (paymentRequest: PaymentRequest) => {
    setIsProcessingPayment(true);

    try {
      const contractKit = ContractKit.newKit("https://alfajores-forno.celo-testnet.org");

      // Get the contract instance
      const contract = await contractKit.contracts.getMyContract();
      const accounts = await contractKit.web3.eth.getAccounts();
      const account = accounts[0];

      // Approve the payment
      await contract.methods
        .approvePayment(paymentRequest.amount * 1e18)
        .send({ from: account})
```

## 10. Install the Payment Gateway on the User Interface: 

We can incorporate the payment gateway into our user interface now that our smart contract has been distributed to the Celo network and our API server is operational. In this stage, we'll utilize React to create a straightforward user interface that enables customers to pay with the Celo stablecoin.

The @celo/dappkit and @celo/contractkit packages must first be installed in our project. Run the following command in the directory for your project:

`npm install @celo/dappkit @celo/contractkit`

The next step is to build a PaymentForm component, which will let users enter the payment amount and start the payment transaction. In the src directory, create a new file called PaymentForm.tsx and add the following code to it:

```javascript
import React, { useState } from 'react';
import { newKitFromWeb3 } from '@celo/contractkit';
import { CeloContract } from '../constants';
import { useCeloDapp } from '../hooks/useCeloDapp';

interface PaymentFormProps {
  recipient: string;
  onSuccess: () => void;
}

export const PaymentForm: React.FC<PaymentFormProps> = ({ recipient, onSuccess }) => {
  const [amount, setAmount] = useState<string>('');
  const { dapp } = useCeloDapp();
  const kit = newKitFromWeb3(dapp.web3);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const stableToken = await kit.contracts.getStableToken();
    const parsedAmount = kit.web3.utils.toWei(amount.toString(), 'ether');
    const txObject = stableToken.transfer(recipient, parsedAmount);
    await dapp.kit.sendTransactionObject(txObject, { from: dapp.address });
    onSuccess();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="number"
        placeholder="Amount"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
      />
      <button type="submit">Pay</button>
    </form>
  );
};
```

Users can enter the amount they wish to pay using the PaymentForm component, which also starts the payment transaction when they click the Pay button. The Celo web3 provider and the kit object, which we use to communicate with the Celo smart contract, are both accessible through the useCeloDapp hook.

The App component will then be updated to render the PaymentForm component. App.tsx's content should be replaced with the following code:

```javascipt
// Import React and PaymentForm component
import React from 'react';
import { PaymentForm } from './components/PaymentForm';

// Define App component as a function component that returns a div containing a PaymentForm
const App: React.FC = () => {
  // Define a recipient address for the payment
  const recipient = '0x0000000000000000000000000000000000000000';

  // Define a callback function to be called when the payment is successful
  const handlePaymentSuccess = () => {
    alert('Payment successful!');
  };

  // Render the App component with a header and a PaymentForm component
  return (
    <div>
      <h1>Celo Payment Gateway</h1>
      <PaymentForm recipient={recipient} onSuccess={handlePaymentSuccess} />
    </div>
  );
};

export default App;
```

When the payment is successful, an alert is displayed and the recipient address is passed to the PaymentForm component as a prop.

Then, in your project directory, use the following command to launch the development server:

`npm start`

## 11. Create the payment component:

Let's develop the Payment component now that the required helper functions have been built. Users will be able to enter the payment amount and start the payment process using this component.

In the src/components directory, make a new file called Payment.tsx and add the following code:

```javascript
import React, { useState } from "react";
import { BigNumber } from "bignumber.js";
import { Contract } from "web3-eth-contract";
import { PaymentProps } from "../types";
import { formatCurrency } from "../utils";

interface PaymentState {
  amount: string;
  loading: boolean;
  error: string;
  success: boolean;
}

const Payment: React.FC<PaymentProps> = ({ account, contract }) => {
  const [state, setState] = useState<PaymentState>({
    amount: "",
    loading: false,
    error: "",
    success: false,
  });

  const handleAmountChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setState((prevState) => ({
      ...prevState,
      amount: e.target.value,
    }));
  };

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setState((prevState) => ({ ...prevState, loading: true }));

    try {
      const amount = new BigNumber(state.amount);
      await contract.methods
        .pay()
        .send({ from: account, value: amount.toString() });

      setState((prevState) => ({
        ...prevState,
        loading: false,
        success: true,
      }));
    } catch (error) {
      setState((prevState) => ({
        ...prevState,
        loading: false,
        error: error.message,
      }));
    }
  };

  return (
    <div className="payment">
      <h2 className="payment__title">Make a Payment</h2>
      <form onSubmit={handleSubmit}>
        <div className="payment__form-row">
          <label className="payment__label" htmlFor="amount">
            Amount
          </label>
          <input
            className="payment__input"
            type="number"
            name="amount"
            id="amount"
            value={state.amount}
            onChange={handleAmountChange}
            required
          />
        </div>
        <button className="payment__button" type="submit">
          {state.loading ? "Processing..." : "Pay"}
        </button>
        {state.error && <p className="payment__error">{state.error}</p>}
        {state.success && (
          <p className="payment__success">Payment successful!</p>
        )}
      </form>
    </div>
  );
};

export default Payment;
```

## 12. Implementing the Payment Confirmation Page:

The user must be directed to a confirmation page where they can view the transaction information after the payment has been handled properly. We will put the payment confirmation page into place in this step.


Let's begin by adding a new file to the pages directory named PaymentConfirmation.tsx. To render the payment confirmation page, we will utilize this file:

```javascript
import React from "react";
import { NextPage } from "next";
import { useRouter } from "next/router";

const PaymentConfirmation: NextPage = () => {
  const router = useRouter();
  const { amount, currency, recipientAddress, txHash } = router.query;

  return (
    <div>
      <h1>Payment Confirmation</h1>
      <p>Amount: {amount} {currency}</p>
      <p>Recipient Address: {recipientAddress}</p>
      <p>Transaction Hash: {txHash}</p>
    </div>
  );
};

export default PaymentConfirmation;
```
The useRouter hook from Next.js is used in this file to retrieve the query parameters from the URL. From the query parameters, we take the amount, currency, recipient address, and transaction hash, and we render these on the page.

## 13. Testing the payment gateway:

Let's test out our Celo payment gateway now that we have integrated all the relevant parts. You may launch the program by typing npm run dev. To view the application's home page, open your browser and navigate to http://localhost:3000.

Click the "Pay with Celo" button after entering the recipient's address, the payment amount, and the currency. This ought to launch the Celo Wallet app on your smartphone.

In the Celo Wallet app, confirm the transaction, then wait for it to be executed. When the transaction is successful, you ought to be taken to the payment confirmation page, where you can view the transaction's specifics.

Congratulations!!! Utilizing React, TypeScript, and the Celo payment gateway, you are done.

## Conclusion: 

Therefore in this tutorial, we learned how to use React, TypeScript, and the Celo SDK to create a Celo payment gateway. The first step was to create a new Next.js project and add the required dependencies. Then, we developed a form that enables users to enter the recipient's address, the payment's amount, and its currency.

After that, we added the Celo SDK to our project and created some codes to start a payment transaction with the ContractKit. To give the user feedback during the payment process, we also added a loading spinner and error messages.

When the transaction has been properly completed, we finally constructed a payment confirmation page where the user can view the specifics of the transaction.

For businesses and customers, creating a Celo payment gateway offers a safe and open payment system. Additionally, it offers a fantastic chance to learn about decentralized finance, smart contracts, and blockchain development.

I appreciate you following along with these instructions to make your coding journey fun!!
