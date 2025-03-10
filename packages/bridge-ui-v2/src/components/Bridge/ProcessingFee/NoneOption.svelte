<script lang="ts">
  import type { Address } from 'abitype';

  import { recommendProcessingFee } from '$libs/fee';
  import { getBalance, type Token } from '$libs/token';
  import { account, network } from '$stores';

  import { destNetwork, selectedToken } from '../state';

  export let enoughEth: boolean;
  export let calculating = false;
  export let error = false;

  async function compute(token: Maybe<Token>, userAddress?: Address, srcChainId?: number, destChainId?: number) {
    if (!token || !userAddress || !srcChainId || !destChainId) {
      enoughEth = false;
      return;
    }

    calculating = true;
    error = false;

    try {
      // Get the balance of the user on the destination chain
      const destBalance = await getBalance({
        token,
        userAddress,
        chainId: destChainId,
      });

      // Calculate the recommended amount of ETH needed for processMessage call
      const recommendedAmount = await recommendProcessingFee({
        token,
        destChainId,
        srcChainId,
      });

      // Does the user have enough ETH to claim manually on the destination chain?
      enoughEth = destBalance ? destBalance?.value >= recommendedAmount : false;
    } catch (err) {
      console.error(err);

      error = true;
      enoughEth = false;
    } finally {
      calculating = false;
    }
  }

  $: compute($selectedToken, $account?.address, $network?.id, $destNetwork?.id);
</script>
