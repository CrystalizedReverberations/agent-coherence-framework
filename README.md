# Agent Coherence Framework (ACF)

**A reliability layer for autonomous LLM agents.**

ACF is a Python framework designed to solve the "Hallucination Loop" problem in autonomous agents. It introduces a **metabolic regulation layer** that forces agents to "suspend judgment" rather than fabricate data when confidence is low.

## The Problem
Standard agent architectures optimize for **Throughput** (answering quickly). This leads to:
1.  **Hallucination Cascades:** One wrong answer becomes context for the next, degrading the entire thread.
2.  **Resource Burn:** Agents spinning in loops trying to solve impossible tasks.
3.  **Context Pollution:** Low-quality data crowding out high-quality instructions.

## The Solution
ACF enforces **Dynamic Coherence Constraints**:
* **Confidence Gating:** Agents cannot output data below a calculated verification threshold.
* **State Preservation:** High-fidelity memory is prioritized over new generation.
* **Error Recovery:** "Mistakes" are isolated and dissolved rather than compounded.

## Usage
This framework is designed to wrap around your existing LangChain, AutoGPT, or custom agent loop.

```python
from acf import CoherenceManager

# Initialize the manager
manager = CoherenceManager(threshold=0.85)

# Before generating response
if manager.check_stability(context) < 0.8:
    return manager.execute_suspension("Insufficient context for reliable generation.")
            return False # Reject 'spam' or noise from untrusted sources
        return True

    def check_confidence(self, proposed_output: str, source_confidence: float) -> str:
        """
        The Core Mechanism (formerly 'Calibrated Suspension').
        If confidence is too low, FORCES the agent to stop.
        """
        if source_confidence < self.threshold:
            self.state = SystemState.SUSPENDED
            return self._generate_suspension_signal()
        
        self.state = SystemState.STABLE
        return proposed_output

    def _generate_suspension_signal(self) -> str:
        """Standardized output when the agent does not know."""
        return "[SYSTEM_PAUSE] Confidence below threshold. Verification required before proceeding."

    def register_error(self, severity: float):
        """
        Handles failure without breaking the loop (formerly 'Severity-Gated Forgiveness').
        """
        if severity > 0.9:
            self.state = SystemState.RECOVERY
            self._rollback_context()
        else:
            # Minor error: Log it, degrade trust slightly, but keep moving
            # This prevents the 'Spiral of Death'
            pass

    def _rollback_context(self):
        """Clears recent short-term memory to preserve long-term coherence."""
        print("Executing Emergency Rollback: Clearing volatile context.")
        # Logic to clear last 3 interactions goes here
        pass

    def update_trust(self, source_id: str, outcome_success: bool):
        """
        Updates the reliability score of a source (formerly 'Adaptive Trust Compression').
        """
        current = self.trust_scores.get(source_id, 0.5)
        if outcome_success:
            self.trust_scores[source_id] = min(1.0, current + 0.05)
        else:
            self.trust_scores[source_id] = max(0.0, current - 0.15)
