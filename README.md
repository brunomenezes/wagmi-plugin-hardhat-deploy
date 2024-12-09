# Wagmi CLI Plugin for hardhat-deploy

This is a [Wagmi CLI plugin](https://wagmi.sh/cli/plugins) that loads contracts and deployments from [hardhat-deploy](https://github.com/wighawag/hardhat-deploy) artifacts generated by the `--export` option of the `deploy` command.

The `deploy --export <path>` option accepts a path and writes a json file with minimal information for interacting with the deployed contracts. The snippet below demonstrates the file format (the abi is truncated for brevity).

```json
{
    "name": "sepolia",
    "chainId": "11155111",
    "contracts": {
        "CartesiDAppFactory": {
            "address": "0x7122cd1221C20892234186facfE8615e6743Ab02",
            "abi": [
                /*...*/
            ]
        }
    }
}
```

Several files, one for each supported network, may be written to a given directory. This plugin loads the contracts and deployments from these files and makes them available to the Wagmi CLI.

The contracts addresses are also imported following [wagmi/cli format](https://wagmi.sh/cli/configuration/options#address-optional), meaning:

-   if there are multiple files (multiple networks), and the addresses are the same in all networks, the address is defined as a single value.

-   if there are multiple files (multiple networks), but the addresses are different, the address is defined as an object with the network id as the key and the address as the value.

> ps: deployment to the same address in multiple networks can be achieved using [deterministic deployment](https://github.com/wighawag/hardhat-deploy#4-deterministicdeployment-ability-to-specify-a-deployment-factory).

## Usage

Given a `hardhat` project the user can use `hardhat-deploy` to deploy to `sepolia` and `mainnet` using the following commands:

```shell
npx hardhat deploy --network sepolia --export export/sepolia.json
npx hardhat deploy --network mainnet --export export/mainnet.json
```

The `wagmi/cli` configuration below will generate a `src/contracts.ts` file containing exported variables (as const) for the ABIs and addresses of each contract.

```typescript
import { defineConfig } from "@wagmi/cli";
import hardhatDeploy from "@sunodo/wagmi-plugin-hardhat-deploy";

export default defineConfig({
    out: "src/contracts.ts",
    plugins: [hardhatDeploy({ directory: "export" })],
});
```

## Configuration

The plugin accepts the following configuration:

-   `directory`: the directory where the `hardhat-deploy` artifacts are located. REQUIRED.

-   `includes`: an array of regular expressions to match the files to be loaded. If not specified, all files in the directory are loaded.

-   `excludes`: an array of regular expressions to match the files to be excluded. If not specified, no files are excluded.

-   `include_networks:` an array of exported filename(s) without the .json extension to include in the output file.
-   `exclude_networks:` an array of exported filename(s) without the .json extension to exclude from the output file.
-   `prefix_contracts:` A string to be prepended to the contract name. Help avoid name collisions and improve intellisense when supporting multiple major versions of the same library.
-   `suffix_contracts:` A string to be appended to the contract name. Help avoid name collisions and improve intellisense when supporting multiple major versions of the same library.

## License

This code is licensed under the Apache 2.0 License. See [LICENSE](./LICENSE) for more information.
