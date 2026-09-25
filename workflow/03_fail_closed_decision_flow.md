# Diagram 3: The Three Possible Verdicts

However confident the AI sounds, only three outcomes are possible, and
they are decided from the actual data, not from the AI's wording.

```mermaid
%%{init: {"theme": "neutral", "themeVariables": {"fontFamily": "Arial, Helvetica, sans-serif", "fontSize": "11px", "actorFontSize": "11px", "noteFontSize": "10px", "messageFontSize": "10px"}}}%%
flowchart TD
    Start["New trail record"] --> Q1{"Can we trust<br/>the location data?"}
    Q1 -->|no| Blocked["Verdict: Blocked<br/>(not enough evidence)"]
    Q1 -->|yes| Q2{"Any active<br/>hazard nearby?"}
    Q2 -->|yes| Caution["Verdict: Caution"]
    Q2 -->|no| Safe["Verdict: Safe"]

```
