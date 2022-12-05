# ftxonsolana
Transaction history for FTX and Alameda-related Solana wallets

## Disclaimer
This data may contain mistakes. Parsers or incorrect/malfunctioning API calls may put data in the wrong place, or omit it entirely. If in doubt, take the transaction signature and verify balance changes directly using on of the multiple block explorers on Solana (solana.fm, or solscan.io are good ones). This data is not comprehensive--it is evolving as fast as I able to ingest it. I will not make any judgments about what any of this data means. It is publicly available, on-chain data, I am merely taking steps in packaging it for those who would like to analyze it. The labeling of any address is a guess and is only provided for clarity; is it not an allegation of any wrongdoing. 

## Motivation
In the wake of the Alameda implosion, and the FTX empire's subsequent bankruptcy, many people have become interested in the organizations' on-chain activity. While there are many skillful bitcoin and ethereum on-chain analysts, and off-the-shelf products, the infrastructure, commercial tools, and general know-how for how to process Solana history is currently less developed. As Solana was the epicenter for FTX/Alameda-supported token launches, it is a reasonable place to start to look. My job here is merely a curator, and I will do my best to tend to the data carefully.

## Summary -- WHAT IT IS
The data here is processed SPL token transactions for known FTX addresses:
1. FTX = 6ZRCB7AAqGre6c72PRz3MHLC73VMYvJ8bi9KHf1HFpNk - <40% complete
2. FTX.us = JBpj7yp4Afvb71TmanVwJZXGeX4kqbGFvjCFCRo3EbTM - 0% complete

At the time, the data set is not complete, as I have only had budget to do about 1.2M of over 3.5M SPL token transactions for the major FTX token addresses, for FTX international only.
In short, there are more, and I will update as I am able to acquire and parse the data. This is just a starting point for those who want to are eager to get visibility into how the exchange interacted with Solana.

## Procedure
At this time FTX has over 40,000 associated token accounts (ATA) on the 6ZRC... address alone. This overwhelms RPC requests to obtain all of the ATAs (e.g. `spl-token accounts --owner 6ZRCB7AAqGre6c72PRz3MHLC73VMYvJ8bi9KHf1HFpNk`), then times out.
So, in a first attempt to get token data what I did was grab the FTX SPL token accounts that sent anything to the recent bankruptcy/recovery/whatever account (6b4aypBhH337qSzzkbeoHWzTLt4DjG2aG8GkrrTQJfQA). From here, I could see what was an associated token account owned by 6ZRCB, which we will consider the "major" FTX SPL accounts for now. Beware that there are many more addresses, and these are currently not included in the data set. Will try to backfill the other ~39900 addresse later, as they will likely emerge from the data provided herein.

So the general process goes:
1. Collect SPL token addresses (as mentioned above, list included)
2. Collect all signatures for these addresses (included)
3. Parse each transaction with calls to Helius as in the following script:
```
const api_key = "<redacted>"

import pkg from 'axios'; 
const axios = pkg;

// parse specific transaction
const url = `https://api.helius.xyz/v0/transactions/?api-key=${api_key}`
const SIGNATURE = process.argv[2];
const parseTransaction = async () => {
  const { data } = await axios.post(url, { transactions: [`${SIGNATURE}`]})
  const txJson = JSON.stringify(data)
  console.log('{ "parsed_tx": ', txJson, '}')
}
parseTransaction()
```
4. Call the script above repeatedly, using GNU parallel on multiple cores to accelerate the activity, for each of the signatures we want to parse.
5. After raw data is obtained, further clean data into the following format, dump to comma-delimited file (.csv):
    - type - TRANSFER, STAKE, WITHDRAW, UNKNOWN 
    - signature - base58 string, transaction signature. *Note: the signature may not be unique, as multiple token accounts may have been modified in a single transaction*
    - slot - bigint, the block slot number for the tx
    - timestamp - bigint, unix integer timestamp for the tx
    - index - integer, if >0 more than one token account was likely modified
    - mint - base58, SPL token address or "mint ID"
    - fromUserAccount - base58, "from" wallet owner address
    - fromTokenAccount - base58, "from" token account address
    - toUserAccount - base58, "to" wallet owner address
    - toTokenAccount - base58, "to" token account address
    - tokenAmount - numeric, amount of tokens (type determined by "mint") that are being transferred 

6. I further added a JSON file that uses the signature as the key, if that format is more desirable.

## Roadmap
1. Get and parse history
2. add coin "mint id" labels for legibility
3. Map addresses labels for Alameda / Alameda-connected wallets to owners and token addresses interfacing w/ FTX 
4. Provide summary data of top flows to/from FTX and connected wallets. Hopefully a network graph can be obtained (if you want to do one, please do it b/c I don't know how)
5. Do SOL transfers/transactions. SOL is not an SPL (just as ETH is not an ERC-20), and so the SOL transactions just roll up to the main address. This presents the problem of trying to query it--it will include most of the SPL transactions, and will be an overwhelming amount of data to process. So the plan is to find all top-level wallet signatures, then remove the set of SPL signatures (that we have here) from those, then processes only the new/unique ones, which should be SOL transfers only. It should be easy to do, but I have not gotten to it yet.

## Verification
To do my best in providing a chain of custody for the data, you will see that each set of files has an included SHA-256 checksum that's in a checksum file (ex: shasums256.txt). The checksum file has a detached GPG signature (shasums256.txt.sig), and the public key is included in the repo. If the tarball checksums do not match, then it's safe to say that they have been modified, and are no longer representative of the work I did. They could be malicious, or merely the data has been altered. Please use caution, verify signatures, and use at your own risk!  

## Contribute
I feel there is definitely a place for data science, AI/ML whizzes to volunteer to this effort. If you feel like you have something to offer, please reach out, or submit a pull request.

## Support
I am currently paying for the API calls out of pocket to Helius, which will be at least $400/mo until the project is complete. If you have found this data helpful, and/or you want to support me continuing on this effort, please consider making a donation to this Solana address: nmsis7BgUJUh881t7YCPYzMRrcJYwnnX62xPU6zxVsb

## Thank you
Thank you in advance for your feedback, effort, and support of this project