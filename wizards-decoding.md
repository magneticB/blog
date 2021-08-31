# Decoding Forgotten Runes Wizard's Cult from on-chain data


Forgotten Runes Wizard's Cult (FRWC) is an NFT project where the data and code used to generate Wizards is stored on-chain directly in the Ethereum blockchain.  This is different from a lot of other NFT projects where usually the  NFT image is stored off-chain in some kind of external filestore (like IPFS).  FRWC allows Wizard images to be generated 100% from code and data stored on-chain.  Wizard NFT images are also stored in IPFS for conviniance (contract can be used to generate this URL) but are not required.

This guide is based on a script from project founder *dotta*.  If you just want [the script](https://gist.github.com/cryppadotta/375dee1903598f5163e2c1d7d3ce9db9) then feel free to jump ahead.  I will try and explain how the script works and the context around it in this guide.

***

####Decoder Crystal

There is a specific assest published by FRWC called the 'Decoding Crystal'.  You can view it [on OpenSea here](https://opensea.io/assets/0x2d00d68bf8bc14d139b4dcea5fb7ce0a42e09c86/0).  As the name implies this is our starting point for generating Wizards.

Under the 'Details' tab on OpenSea you will see the Contract Address field.  Click on it and you can [view the contract on Etherscan](https://etherscan.io/address/0x2d00d68bf8bc14d139b4dcea5fb7ce0a42e09c86) to see the details and source code for the contract.

The decoder code is stored inside the the second transaction on the contract (0x0b8eb29d7a592023b5330fd9a93299bca2a9604aaa2494c87333fc56da50ec9e).  Open [that transaction](https://etherscan.io/tx/0x0b8eb29d7a592023b5330fd9a93299bca2a9604aaa2494c87333fc56da50ec9e) and expand the fields list until you can see the 'Input Data' field.  This is the hex encoded transaction input data which contains the code we are looking for.  Chage the 'View Input As' to 'UTF-8' in the Etherscan website, and the decoder source will magicaly appear.  It's TypeScript and does not run directly on-chain, but it is stored there.

You'll notice that the first few characters of the input data do not covert from hex to UTF-8 into anything readable.  Copy the source code starting from the first comment symbol '//' ignoring the first few characters. Save them into a 'decoder.ts' file using your favorite text editor.

If you're following along with the script provided by *dotta* you'll notice the first line is creating the same decoder.ts file.  It's doing the same as thing we did manually above in these steps:
>1. Downloads the transaction data over HTTP (note the same transation id).  https://cloudflare-eth.com/ is used to get the data and return in JSON format.  Both https://cloudflare-eth.com/ and Etherscan are interfaces which get their data directly from the Ethereum blockchain.
>2. [jq](https://stedolan.github.io/jq/) then extracts the input data from the tranaction JSON response.  This is hex encoded.
>3. cut removes the first few characters.
>4. xxd converts the hex encoded data into binary
>5. head trims off the last line and pipes the output into 'decoder.ts' file.

You might need to install some dependencies (like jq) to run the script.  Using either method you are basically converting the transaction input data and saving the result to a file.

***

####Compiling and running the script

The decoder.ts script runs with [node.js](https://nodejs.org/) and requires some dependencies.  Make sure node.js is already installed and path set correctly before trying this (I'm using node v14.17).

1. Setup a new node project in the same directory where decoder.ts is stored:

	><code>npm init -y</code>

2. Download and install the required dependencies to run the script including TypeScript:

	><code>npm install ethers@5.0.26 yargs@16.1.0 chalk@4.1.0 ora@5.3.0 ts-node@9.0.0 typescript@4.0.5 bson@4.4.0 sharp@0.28.3 parse-numeric-range@1.2.0 mkdirp@1.0.4 @types/yargs @types/node</code>

3. Run the script with the TypeScript node dependency:

	><code>./node_modules/.bin/ts-node ./decoder.ts --wizards "7934,101-103"</code>

This should run without errors and you shound find your wizards created inside a 'wizards' sub directory.  You can change the ids passed in the 'wizards' argument to any of the 10,000 wizards.

*** 
####Script details

There are three main functions in the script which we will talk about.  There is a 'provenance' variable that stores several ethereum transaction hashes.  These transactions store the Wizards parts image and trait data inside the transaction input data.  Same method as how the decoder source was stored just using different transations.

#####decodeParts()
This function uses the [Ethers](https://ethers.org/) project to download the 'provenance.img' transation.  The [transaction input data](https://etherscan.io/tx/0xbb6413bd70bae87b724c30ba9e46224fa63629709e7ccfe60a39cc14aa41013e) for this transaction (found in provenance.img) stores a PNG image on-chain which is then extracted and saved to an image file locally.  You can see the PNG header if you switch the 'View Input As' to UTF-8 on Etherscan.

The image file is saved to 'forgotten-runes-traits.png' inside the 'wizards' directory.  Open it up and you will see the building blocks of every wizard.  In a later step these are composed together to generate each wizard.


//Todo
