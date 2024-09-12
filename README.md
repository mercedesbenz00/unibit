## Features

- Wallet connectivity using [Onboard](https://onboard.blocknative.com/).
- Supported networks and UI indicators when the network is not supported.
- TokenInput with validations [src/components/token/TokenInput.tsx](src/components/token/TokenInput.tsx)
- Spend token [src/components/token/TokenSpend.tsx](src/components/token/TokenSpend.tsx)
- Configurable Token list modal [src/components/token/TokenModal.tsx](src/components/token/TokenModal.tsx)
- Auto-generated types for smart contracts.
- Transactions Life cycle. With homemade toasts.
- Support for connecting with subgraphs.
- Typed hooks to read data from smart contracts.
- Typed hooks to call methods of smart contracts.
- Web3ConnectionProvider: it has many helpers that let you know about the status of a wallet connection.
- React suspense.
- Fetching data through [useSWR](https://swr.vercel.app/).
- Basic layout with support for mobile (menu, sidebar, content).
- Cookie policy banner.
- Many UI components have been created with [styled-components](https://styled-components.com/).
- Light and dark mode.

## Getting Started

You can fork this repo using Github templates. Here is an explanation of how to do it [creating-a-repository-from-a-template](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template#creating-a-repository-from-a-template).

## Installation

1. Install dependencies

```bash
yarn install
```

2. Make a copy of [`.env.example`](./.env.example) to `.env.local` and assign values to the variables

```bash
cp .env.example .env.local
```

> Supported providers are [Alchemy](https://www.alchemy.com/) and [infura](https://infura.io/).

3. Run the app. It will be available at `http://localhost:3000`.

```bash
yarn dev
```

## File structure

This starter is based on [nextjs](https://nextjs.org/docs/getting-started).

Inside the `src` folder you will find:

- `components`: Stateless components. Mostly focused on UI.
- `constants`
- `contracts`: All the contracts your dApp will interact with have to be configured here.
- `hooks`: React hooks.
  - `queries`: Hooks that retrieve data from the blockchain (or api). _shouldn't be used directly from inside a Component_
  - `presentation`: Hooks that transform and combine data to show in the UI. _Should be used in components_
  - `mutations`: Hooks that provides a mutation to write data on the blockchain (or api). _Should be used in components_
- `pagePartials`: This folder was thought to be used as a complement to the files in `pages` folder. Sometimes a page can become big and this is the place where helper components related to a particular page will reside. We suggest having one folder per page.
- `providers`: React providers.
- `subgraph`: Queries and configuration for Subgraphs.
- `theme`: general UI styles.
- `utils`: Some utility functions.
- `config`: Global configurations.

## How it works

The main entry point of this starter is the [`src/config/web3.ts`](./src/config/web3.ts) file, where you can set up the supported chains.

```ts
export const Chains = {
  mainnet: 1,
  goerli: 5,
} as const
```

Then, you can start adding the smart-contract you want to interact with. To do so, you need to edit this file [`src/contracts/contracts.ts`](./src/contracts/contracts.ts). Note that you will need to get the ABI. If a new contract is added, you will need to run this script `yarn types-gen`, it will generate the Typescript types for the ABI.

Once your contracts are configured, you can make use of the following react hooks.

- `useContractCall`: will allow you to call a smart-contract method or methods, it is typed, so it will let you know the arguments of the methods as well as the response it returns.

```ts
// first, you need to import the types from the auto-generated types.
import { ERC20, ERC20__factory } from '@/types/typechain'

// create a contract instance. Note that the second parameter is the name we have assigned when we defined the smart-contract in `src/constants/config/contracts.ts`
const erc20 = useContractInstance(ERC20__factory, 'USDC')

// create an array with the methods selectors you want to call.
const calls = [erc20.balanceOf, erc20.decimals] as const

// make use of useContractCall to call them.
const [{ data }, refetch] = useContractCall<ERC20, typeof calls>(calls, [
  [address || ZERO_ADDRESS],
  [],
])

// data will be typed.
const [usdcBalance, usdcDecimals] = data || [ZERO_BN, 18]

const sendTx = useTransaction()

const tx = erc20.transfer(addressToSend, valueToSend)

try {
  const transaction = await sendTx(tx)
  const receipt = await transaction.wait()
  console.log(receipt)
} catch (error) {
  console.error(error)
}
```

we also provide a Button helper that works like so:

```tsx
<TxButton
  disabled={disableSubmit}
  onMined={(r) => {
    // do something
  }}
  onSend={(tx) => tx && clearForm()}
  tx={() => erc20.transfer(addressToSend, valueToSend)}
>
  Send
</TxButton>
```

## How to add a new smart contract

1. Put your contract ABI file in [`src/contracts/abis`](./src/contracts/abis) as a `.json` file.
2. Go to [`src/contracts/contracts.ts`](./src/contracts/contracts.ts).
3. Provide a key name for your contract in `const contracts = {...}`.
4. Provide a contract address for all the chains you have configured.
5. Import your contract ABI file and add it to the contract key created in step 3.

Example for an ERC20 contract (USDC):

```ts
import ERC_20_abi from '@/src/abis/ERC20.json'

export const contracts = {
  // Other contracts
  USDC: {
    address: {
      [Chains.mainnet]: '0x123...',
      [Chains.goerli]: '0x456...',
    },
    abi: ERC_20_abi,
  },
} as const
```

## Subgraph consumption

To interact with a subgraph you should follow these steps:

1. Go to [`src/subgraph/subgraph.ts`](./src/subgraph/subgraph.ts) and a new entry to the `enum subgraphName`.
2. Go to [`src/subgraph/subgraph-endpoints.json`](./src/subgraph/subgraph-endpoints.json) and complete the object following the structure `{ chain: [name: endpoint] }`.

```json
{
  "5": {
    "subgraphName": "subgraph endpoint"
  }
}
```

3. Create queries for the subgraph in the folder [`src/queries`](./src/subgraph/queries). Remember that we already have a [`src/subgraph/queries/examples.ts`](./src/subgraph/queries/examples.ts) that you can use as a guide.
4. run `yarn subgraph-codegen`. This step has to be done every time you add or modify a query.
5. Consume now you will have all the queries typed and available to be called.

```ts
import { SubgraphName, getSubgraphSdkByNetwork } from '@/src/constants/config/subgraph'

const { appChainId } = useWeb3Connection()

const gql = getSubgraphSdkByNetwork(appChainId, SubgraphName.Rentals)
const res = gql.useSubgraphErrors()
console.log({ res: res.data?._meta }) // res is typed 😎
```

```tsx
export const Paragraph = styled.p`
  color: ${({ theme: { colors } }) => colors.textColor}; // from light.ts or dark.ts
  font-family: ${({ theme: { fonts } }) => fonts.family}; // from common.ts
  font-size: 1.5rem;
  font-weight: 400;
`
```

## Tokens

We use [Token Lists](https://tokenlists.org/) to provide a list of tokens to the app. You can find the map of tokens sources in [`src/config/web3.ts`](./src/config/web3.ts), at `TokensLists`. You can add a new token source to the map by following the same structure as the other lists.

```ts
export const TokensLists = {
  '1INCH': 'https://gateway.ipfs.io/ipns/tokens.1inch.eth',
  COINGECKO: 'https://tokens.coingecko.com/uniswap/all.json',
  OPTIMISM: 'https://static.optimism.io/optimism.tokenlist.json',
} as const
```

To properly support the token images source, you need to whitelist the domains in [`next.config.js`](./next.config.js), at `images.domains`:

```ts
images: {
  domains: ['tokens.1inch.io', 'ethereum-optimism.github.io', 'assets.coingecko.com']
}
```

To consume them, we implemented the hook [`useTokensLists`](./src/hooks/useTokensLists.tsx). You can find a usage example in [`src/components/token/TokenDropdown.tsx`](./src/components/token/TokenDropdown.tsx). And see it in action in our example page.

## Protocol Data

### useMarketsData

filepath: `src/hooks/presentation/useAgaveMarketsData.tsx`

React hook that can be used to get Agave markets information from an array of tokens addresses (reserve tokens addresses). The hook accepts an array of tokens addresses and returns data for each token as `marketData`, such as `priceData`, `reserveData`, `assetData`, and `incentiveData`. Additionally, the hook provides a number of functions that can be used to get data about a single market from the result, such as `getMarketSize`, `getTotalBorrowed`, `getDepositAPY`, `getBorrowRate`, and `getIncentiveRate`.

## Contributing
