🩺 WalletHealthExecutorAgent – ROMA Integration Guide

The WalletHealthExecutorAgent analyzes a Solana wallet’s balances, token holdings, and recent transactions using Helius and Solscan APIs.
This guide explains how to integrate and run it inside the ROMA framework.

📖 Table of Contents

Step 1. Configure Environment

Step 2. Add the Adapter

Step 3. Register the Agent

Step 4. Add Executor Prompt

Step 5. Update Deep Research Agent Profiles

Step 6. Compose Docker

Step 7. Notes

📌 Step 1. Configure Environment

Add the following to your .env file:

HELIUS_API_KEY=your_helius_api_key_here
SOLSCAN_API_KEY=your_solscan_api_key_here
RPC_URL=https://api.mainnet-beta.solana.com
LOG_LEVEL=INFO

📌 Step 2. Add the Adapter

Edit the file:
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agents/adapters.py

and add your custom class:

class WalletHealthAdapter:
    """
    Adapter that analyzes a wallet’s balances, token holdings, 
    and recent transactions using Helius + Solscan APIs.
    """

    def __init__(self, wallet_address: str, helius_api_key=None, solscan_api_key=None):
        self.wallet_address = wallet_address
        self.helius_api_key = helius_api_key or os.getenv("HELIUS_API_KEY")
        self.solscan_api_key = solscan_api_key or os.getenv("SOLSCAN_API_KEY")

    def run(self):
        # Example placeholder – expand with actual API logic
        return {
            "wallet": self.wallet_address,
            "status": "healthy",
            "balances": {"SOL": "2.34"},
            "recent_activity": "3 swaps in last 24h"
        }

📌 Step 3. Register the Agent

In
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/agents.yaml,
add:

- name: "WalletHealthExecutor"
  type: "executor"
  adapter_class: "sentientresearchagent.hierarchical_agent_framework.agents.adapters.WalletHealthAdapter"
  description: "Fetches wallet balances, token holdings, and recent transactions from Helius + Solscan"

  model:
    provider: "litellm"
    model_id: "openrouter/google/gemini-2.5-flash:nitro"

  prompt_source: "prompts.executor_prompts.WALLET_HEALTH_EXECUTOR_SYSTEM_MESSAGE"

  registration:
    action_keys:
      - action_verb: "execute"
        task_type: "WALLET_HEALTH"
    named_keys: ["WalletHealthExecutor", "wallet_health_checker"]

  enabled: true

📌 Step 4. Add Executor Prompt

Create or edit:
ROMA/src/sentientresearchagent/hierarchical_agent_framework/prompts/executor_prompts.py

Add:

WALLET_HEALTH_EXECUTOR_SYSTEM_MESSAGE = f"""You are a Solana wallet health analyzer.

### Objectives:
1. Fetch balances, token holdings, and transactions for the target wallet.
2. Summarize the wallet’s health in plain English.
3. Highlight:
   - Current SOL balance
   - Top token holdings
   - Recent swap or transfer activity
   - Risk level if wallet is inactive or draining

### Response format:
Always return:
- Wallet Address
- SOL Balance
- Top Tokens
- Recent Activity Summary
- Health Status
"""

📌 Step 5. Update Deep Research Agent Profiles

Edit:
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/profiles/deep_research_agent.yaml

Replace:

executor_adapter_names:
  SEARCH: "OpenAICustomSearcher"


with:

executor_adapter_names:
  WALLET_HEALTH: "WalletHealthExecutor"

📌 Step 6. Compose Docker

Rebuild and restart the containers:

docker compose down
docker compose up -d --build

📌 Step 7. Notes

✅ Ensure both HELIUS_API_KEY and SOLSCAN_API_KEY are valid.

✅ Use fresh wallets with activity to see meaningful results.

✅ Extend WalletHealthAdapter.run() to fetch real data instead of placeholders.

⚠️ Helius free tier may have rate limits → consider retries or upgrading.
