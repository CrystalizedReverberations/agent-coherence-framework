
```python
import time
from typing import Dict, List, Any, Optional
from dataclasses import dataclass
from enum import Enum

class SystemState(Enum):
    """Operational states of the agent."""
    STABLE = "stable"              # Fully coherent
    SUSPENDED = "suspended"        # Paused to verify
    DEGRADED = "degraded"          # Low confidence, requires external input
    RECOVERY = "recovery"          # Rolling back to last known good state

@dataclass
class InteractionEvent:
    """A single unit of interaction (formerly 'Signal')."""
    source_id: str
    content: str
    confidence_score: float
    timestamp: float

class CoherenceManager:
    """
    Main controller for Agent Reliability.
    Prevents hallucination by enforcing confidence thresholds.
    """
    
    def __init__(self, confidence_threshold: float = 0.85):
        self.threshold = confidence_threshold
        self.state = SystemState.STABLE
        self.interaction_history: List[InteractionEvent] = []
        self.trust_scores: Dict[str, float] = {}

    def assess_input(self, input_data: str, source_id: str) -> bool:
        """
        Filters incoming data (formerly 'Selective Permeability').
        Rejects input that increases system entropy too fast.
        """
        current_trust = self.trust_scores.get(source_id, 0.5)
        
        # Simple heuristic: If trust is low, we need higher verification
        if current_trust < 0.4 and len(input_data) > 1000:
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
