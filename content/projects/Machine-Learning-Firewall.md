```markdown
---
title: "Building an Active Machine Learning Application Firewall (MLAF)"
draft: false
tags: ["AI Security", "Adversarial ML", "Python", "FastAPI", "Offensive Security", "Docker"]
categories: ["Projects"]
summary: "An inline, active-defense proxy neutralizing adversarial ML payloads before they reach production inference APIs."
cover:
  image: "" # Add a screenshot of your terminal blocking the exploit here!
  alt: "MLAF Architecture"
  caption: "Defending ML pipelines against decision-based evasion attacks."
---

## The Threat: When Valid JSON is a Weapon

Everyone is rushing to deploy AI models, but the inference pipelines are being left wide open. 

Traditional Web Application Firewalls (WAFs) are blind to adversarial machine learning attacks. If an attacker uses a decision-based black-box technique—like a HopSkipJump attack—to subtly perturb input data, a standard WAF sees perfectly formatted, valid JSON and passes it right through. To the financial fraud model or the safety filter on the other side, however, those microscopic mathematical shifts completely break the decision boundary.

If an evasion attack bypasses a fraud model, the business absorbs the loss.

To solve this at the architectural level, I built an inline **Machine Learning Application Firewall (MLAF)**. It operates as an asynchronous FastAPI reverse proxy, neutralizing adversarial ML payloads in real-time without adding crippling latency to the pipeline.

---

## 🏗️ Architecture & Isolation

The core philosophy of this defense is absolute isolation. The vulnerable ML model should never be exposed to the public internet. 

Using Docker Compose, the environment is split into two services:
1.  **The Vulnerable Target (`ml-target`):** A FastAPI service running the Scikit-Learn fraud detection model, exposed only on an internal Docker network.
2.  **The Inline Firewall (`mlaf-proxy`):** The public-facing entry point that intercepts, inspects, and scrubs all traffic before forwarding it to the target.

---

## 🛡️ Active Defense Mechanisms: The Code

To stop optimization-based attacks, the firewall actively modifies the payload. Here is how the `MLFirewall` class neutralizes threats:

### 1. Input Sanitization (Blocking Scalar Exhaustion)
Attackers often try to break models by passing `NaN` or `Infinity`, or by sending extreme outliers. The firewall enforces strict mathematical integrity before model ingestion:

```python
@staticmethod
def sanitize_input(data_array: np.ndarray) -> np.ndarray:
    if not np.isfinite(data_array).all():
        raise ValueError("Payload contains NaN or Infinity.")
    return np.clip(data_array, a_min=-100.0, a_max=100.0)

```

### 2. Feature Squeezing (Snapping Gradients)

Adversarial attacks rely on continuous, microscopic changes to find the exact edge of a model's decision boundary. By aggressively rounding the input data, we "squeeze" the feature space, snapping the subtle gradients the attacker is trying to ride.

```python
@staticmethod
def limit_distribution(data_array: np.ndarray) -> np.ndarray:
    """Aggressive integer rounding to snap evasion attacks."""
    return np.round(data_array, decimals=0)

```

### 3. Gradient Starvation (Output Masking)

If an attacker can see the exact confidence score (e.g., `0.874932`), they can map the proprietary model. The proxy intercepts the egress traffic and blurs the probabilities, starving the attacker of the precision they need to optimize their next payload.

```python
@staticmethod
def mask_output(response_data: dict) -> dict:
    if "confidence_scores" in response_data:
        scores = response_data["confidence_scores"]
        response_data["confidence_scores"] = {
            "legitimate": round(scores.get("legitimate", 0), 2),
            "fraud": round(scores.get("fraud", 0), 2)
        }
    return response_data

```

---

## ⚔️ The Offensive Perspective: Writing the Exploit

You cannot effectively defend a system unless you know exactly how to exploit it.

To validate the MLAF, I wrote a live exploit script (`attack_evasion.py`) utilizing the Adversarial Robustness Toolbox (ART). The script generates a HopSkipJump adversarial transaction against a standard payload.

When firing the exploit directly at the unprotected model, the numbers barely change, but the model flips its prediction:

> **Exploit Successful: Fraud filter bypassed.**

However, when routing the exact same attack through the MLAF on port 8000, the Feature Squeezing normalizes the perturbed data, the attack fails, and the fraud is accurately detected and blocked. The proxy generates actionable telemetry for the SOC, noting the anomaly rather than just returning a generic 403.

---

## Conclusion

Deploying ML models requires more than just wrapping a `.joblib` file in a FastAPI endpoint. By applying an offensive cybersecurity mindset to application architecture, we can build robust, active defenses that protect the business logic from adversarial manipulation.

**🔗 [View the full source code and Docker setup on GitHub**](https://www.google.com/search?q=https://github.com/lakshyarastogi25/machine-learning-application-firewall)

