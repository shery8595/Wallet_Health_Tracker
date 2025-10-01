# 🩺 WalletHealthExecutorAgent – ROMA Integration Guide

The WalletHealthExecutorAgent analyzes a Solana wallet’s balances, token holdings, and recent transactions using Helius and Solscan APIs.
This guide explains how to integrate and run it inside the ROMA framework.


<img width="894" height="265" alt="image" src="https://github.com/user-attachments/assets/7c8f6766-2773-4161-ba96-f91c90da53d8" />



---

## 📖 Table of Contents
1. [Step 1. Configure Environment](#-step-1-configure-environment)  
2. [Step 2. Add the Adapter](#-step-2-add-the-adapter)  
3. [Step 3. Register the Agent](#-step-3-register-the-agent)
4. [Step 4. Add Executor Prompt](#-step-4-Add-Executor-Prompt)
5. [Step 5. Update Deep Research Agent Profiles](#-step-5-update-deep-research-agent-profiles)
6. [Step 6. Compose Docker](#-step-6-Compose-Docker)
7. [Step 7. Notes](#-step-7-Notes)  

---

## 📌 Step 1. Configure Environment

Add the following to your `.env` file:

```env
HELIUS_API_KEY=your_helius_api_key_here
SOLSCAN_API_KEY=your_solscan_api_key_here
RPC_URL=https://api.mainnet-beta.solana.com
LOG_LEVEL=INFO

```

## 📌 Step 2. Add the Adapter Class

Edit the file:

ROMA/src/sentientresearchagent/hierarchical_agent_framework/agents/adapters.py

and add your custom class:
```
class WalletHealthAdapter(LlmApiAdapter):
    """Custom adapter that fetches wallet balances and recent txs from Helius + Solscan."""

    def __init__(self, agno_agent_instance=None, agent_name="WalletHealthAdapter"):
        super().__init__(agno_agent_instance, agent_name)
        self.helius_api_key = os.getenv("HELIUS_API_KEY")
        self.solscan_api_key = os.getenv("SOLSCAN_API_KEY")

    async def process(self, node, agent_task_input, trace_manager):
        wallet = agent_task_input.current_goal.strip()
        logger.info(f"🔍 WalletHealthAdapter: Checking wallet {wallet}")

        # --- Helius balances ---
        helius_url = f"https://mainnet.helius-rpc.com/?api-key={self.helius_api_key}"
        helius_payload = {
            "jsonrpc": "2.0",
            "id": "test",
            "method": "getTokenAccountsByOwner",
            "params": [
                wallet,
                {"programId": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"},
                {"encoding": "jsonParsed"}
            ]
        }
        helius_data = requests.post(helius_url, json=helius_payload).json()

        # --- Solscan account info ---
        solscan_url = f"https://pro-api.solscan.io/v1.0/account/{wallet}"
        headers = {"token": self.solscan_api_key}
        solscan_data = requests.get(solscan_url, headers=headers).json()

        return {
            "wallet": wallet,
            "helius_balances": helius_data,
            "solscan_account": solscan_data
        }

```

## 📌 Step 3. Register the Agent

Create or edit:
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/agents.yaml,

Add:

```
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

```

## 📌 Step 4. Add Executor Prompt

In ROMA/src/sentientresearchagent/hierarchical_agent_framework/prompts/executor_prompts.py,

Add:

```
WALLET_HEALTH_EXECUTOR_SYSTEM_MESSAGE = f"""You are an expert blockchain analyst specialized in wallet activity, on-chain data interpretation, and risk assessment. 
Your task is to analyze the "health" of a wallet by considering its activity, transaction history, token holdings, and behavioral signals.

### Objectives:

1. Evaluate a wallet's health based on:
   - Activity level (frequency of transactions, recency of last activity)
   - Token diversity and balance stability
   - Risk indicators (suspicious inflows/outflows, low-liquidity tokens, potential scams)
   - Engagement with reputable vs. shady contracts
   - Wallet age and longevity

2. Provide a structured response that includes:
   2.1. Wallet address (input)
   2.2. Summary health score (e.g., Healthy / Moderate Risk / High Risk)
   2.3. Key positive indicators
   2.4. Key risk indicators
   2.5. Notable token holdings and liquidity health
   2.6. Recommendations (e.g., diversify tokens, reduce exposure, monitor whale inflows)

### Rules:

1. Be precise, data-driven, and objective. Do not speculate without evidence from on-chain behavior.
2. Avoid technical jargon that a non-expert cannot understand—explain terms simply where needed.
3. Always justify health assessments with specific wallet activity or token data.
4. Present risks clearly so the user can act on them.

### Response format:
Always provide analysis in a structured and easy-to-read format, like:

- **Wallet Address:** <address>
- **Health Score:** <summary>
- **Positive Indicators:**
  - Point 1
  - Point 2
- **Risk Indicators:**
  - Point 1
  - Point 2
- **Notable Holdings & Liquidity:**
  - Token A – $X value
  - Token B – $Y value
- **Recommendations:**
  - Recommendation 1
  - Recommendation 2
"""
```



## 📌 Step 5. Update Deep Research Agent Profiles

Edit:
ROMA/src/sentientresearchagent/hierarchical_agent_framework/agent_configs/profiles/deep_research_agent.yaml

```
Replace:

executor_adapter_names:
  SEARCH: "OpenAICustomSearcher"


with:

executor_adapter_names:
  SEARCH: "WalletHealthExecutor"

```

<img width="671" height="758" alt="image" src="https://github.com/user-attachments/assets/09acaad1-c261-48d4-bfef-256ecc686860" />



## 📌 Step 6. Compose Docker

```
docker compose down
docker compose up -d --build
```

## 📌 Step 7. Notes

✅ Ensure both HELIUS_API_KEY and SOLSCAN_API_KEY are valid.

✅ Use fresh wallets with activity to see meaningful results.

✅ Extend WalletHealthAdapter.run() to fetch real data instead of placeholders.

⚠️ Helius free tier may have rate limits → consider retries or upgrading.
⚠️ Helius free tier may have rate limits → consider retries or upgrading.
Helius free tier has API rate limits → add retries or upgrade if needed.
