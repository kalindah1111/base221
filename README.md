# base221import time
from collections import defaultdict
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

WINDOW_BLOCKS = 20
RECIPIENT_THRESHOLD = 10  # number of unique recipients


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))

    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Tracking multi-recipient senders...\n")

    last_block = w3.eth.block_number

    while True:
        try:
            current_block = w3.eth.block_number

            if current_block >= last_block + WINDOW_BLOCKS:

                from_block = current_block - WINDOW_BLOCKS
                to_block = current_block

                sender_recipients = defaultdict(set)

                for b in range(from_block, to_block + 1):

                    block = w3.eth.get_block(b, full_transactions=True)

                    for tx in block.transactions:

                        if tx["to"]:
                            sender_recipients[tx["from"]].add(tx["to"])

                print(f"\nBlocks {from_block} → {to_block}")

                for sender, recipients in sender_recipients.items():
                    if len(recipients) >= RECIPIENT_THRESHOLD:
                        print("📤 Multi-Recipient Sender")
                        print("Sender:", sender)
                        print("Unique recipients:", len(recipients))
                        print()

                last_block = current_block

            time.sleep(3)

        except Exception as e:
            print("Error:", e)
            time.sleep(5)


if __name__ == "__main__":
    main()
