# Solana Testnet Cluster Restart 21 February 2024

Block production on Solana Testnet has halted at approximately 16:30 UTC on 21 February 2024.

## Confirm highest optimistic slot:

**solana-ledger-tool -l ledger latest-optimistic-slots**

Note: If your last confirmed slot is lower than that the one listed above, this is likely your node crashed before it was able to observe the latest supermajority. In this case, update according to step 2 then proceed to the Appendix.

Important: DO NOT delete your ledger directory.

## Step 1: Create a snapshot at slot 254108256
You need to stop your validator process if it is still running.

This document assumes your ledger directory is called ledger/.  If not then adjust the following commands accordingly.

Use the ledger tool to create a new snapshot at slot 254108256, replacing the two instances of <ledger path> to your actual ledger path:

```
solana-ledger-tool -l <ledger-path> \
--snapshot-archive-path <snapshot-path> \
--incremental-snapshot-archive-path <incremental-snapshot-path> \
create-snapshot 254108256 <snapshot-path> \
--hard-fork 254108256 \
--deactivate-feature-gate EJJewYSddEEtSZHiqugnvhQHiWyZKjkFDQASd7oKSagn
```
 
If you have a custom accounts path add `--accounts <PATH_TO_ACCOUNTS> \` before `--hard-fork`

The final line of output should be 

```
Successfully created snapshot for slot 254108257, hash 4rWEDhTyQVgTw6sPoCthXmUNmjeiwsdKQ5ZNvpEi3uvk: /mnt/ledger/snapshot-254108257-ABaHfFrncb3Y9KXHJu25uV3McaqDda2xhWdUCtqmaSEe.tar.zst
Shred version: 35459
``` 

## Step 2: Adjust your validator command-line arguments, temporarily for this restart to include:
(--known-validators aren’t needed if you have your own local snapshot and have set –no-genesis-fetch as your validator won’t be downloading anything, you can omit those arguments in this case)

**If you use a have a hot failover setup ensure you start with the correct identity!**

```
--wait-for-supermajority 254108257 \
--no-snapshot-fetch \
--no-genesis-fetch \
--expected-bank-hash 4rWEDhTyQVgTw6sPoCthXmUNmjeiwsdKQ5ZNvpEi3uvk \
--expected-shred-version 35459 \
```

(Remove the previous value of “--expected-shred-version“ if present). 

Once the cluster restarts and normal operation resumes, remember to remove --wait-for-supermajority and --expected-bank-hash before the next update or restart. They are only required for the restart. You can also go back to your old known-validators at that point.

## Step 3: Update your validator version

N/A - no patch at this time

## Step 4: Start your validator and verify status
As it boots, it will load the snapshot for slot 254108257 and wait for 80% of the stake to come online before producing/validating new blocks. 

To confirm your restarted validator is correctly waiting for 80% stake, look for this periodic log message to confirm it is waiting:
INFO  solana_core::validator] Waiting for 80% of activated stake at slot 246464040 to be in gossip...

And if you have RPC enabled, ask it repeated for the current slot:
```
solana --url http://127.0.0.1:8899 slot
```

Verify that you have the correct identity configuration with this command
```
solana-validator -l /PATH/TO/LEDGER contact-info
```

Any number other than 254108257 means you did not complete the steps correctly.

Once started you should see log entries for “Activate stake” visible in gossip and “waiting for 80% of stake” to be visible. You can track these to see how stake progresses.

If you couldn’t produce your snapshot locally follow appendix on next page below 



## Appendix: Resolution if you did not preserve your ledger or your last optimistically confirmed slot is below 254108256

NOT RECOMMENDED - this resolution should only be attempted if your ledger/ directory is unavailable or you are unable to produce a snapshot for 254108257.

ONLY IF your ledger history is corrupt or otherwise unavailable and your last confirmed slot is lower than 254108257, follow these instructions to get a new snapshot:

Your validator will need to download a new snapshot from one of the known validators. Alternative snapshot download methods are also provided further below. A snapshot will be verified as valid by the bank hash in the arguments below. 

For this you need to remove –no-snapshot-fetch if present.

Add these arguments to restart:
```
 --wait-for-supermajority 254108257 \
 --expected-shred-version 35459 \
 --expected-bank-hash 4rWEDhTyQVgTw6sPoCthXmUNmjeiwsdKQ5ZNvpEi3uvk \
```

## Additional snapshot sources:
If you follow this add back –no-snapshot-fetch, you can remove old snapshots, none should have a higher slot than the one you download.

You can use these known validators:

```
```
