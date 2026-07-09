# AI Prompt Ledger & Analysis
**Student ID:** 20231000096  
**Course:** Advanced Object-Oriented Programming & Systems Design  

---

## 1. Buffer Leak (Encapsulation Flaw)

### Exact Engineering Prompt
> "Analyze TelemetryBuffer.__init__ in Python. The parameter `frame_buffer: list = []` causes all instantiated sensor objects to store data inside the exact same underlying array reference, resulting in a hardware memory overflow fault. Provide a robust object encapsulation fix."

### Why the AI's Initial Suggestion Failed
The AI originally recommended clearing the list using `self.frame_buffer.clear()` inside the constructor or using a deepcopy. In an embedded context, clearing the list inside the constructor destroys structural records belonging to previously running tasks sharing that scope, and deepcopying adds unnecessary runtime computational overhead to memory-constrained microcontrollers.

### Computer Engineering Justification
The default value parameter `list = []` evaluates once during runtime module evaluation, binding a single mutable list reference across all objects. By switching the signature default to `None` and instantiating an empty list reference `[]` dynamically within the local memory bounds of the instance, unique hardware channels are cleanly encapsulated in independent heap frames.

---

## 2. Polymorphic Trap (Liskov Substitution Principle Violation)

### Exact Engineering Prompt
> "Review IMUPeripheral.poll_raw_voltage(). When the seed-injected drift factor exceeds 2.0, it breaks the base class contract by returning a dictionary error object instead of a floating-point voltage. Standardize this method to adhere to the Liskov Substitution Principle."

### Why the AI's Initial Suggestion Failed
The AI initially suggested changing the base class contract type hint from `float` to `Union[float, dict]`. In real-world flight control systems, modifying processing registers to accept data type unions is highly dangerous; it forces downstream signal pipelines to perform type checking, which adds CPU execution lag and crashes math operations during signal scaling.

### Computer Engineering Justification
To fulfill the Liskov Substitution Principle (LSP), a derived subclass must be entirely swappable with its parent class without breaking system execution. Standardizing the error state block to push its debug frames to the buffer but ultimately return a uniform float data type (`0.0`) keeps the data pipeline safe without mutating downstream execution logic.

---

## 3. Race Condition (Concurrency Flaw)

### Exact Engineering Prompt
> "A race condition occurs inside the AvionicsBusMaster read-modify-write block on `self.active_bus_register` because multiple threads access it simultaneously while an artificial time sleep forces task switching. Implement an atomic synchronization lock using threading.Lock."

### Why the AI's Initial Suggestion Failed
The AI suggested using a simple global boolean flag status variable (like `busy = True`) to track active updates. This fails in low-level multitasking engines because check-and-set routines on standard flags are not atomic themselves; multiple threads can check the flag simultaneously, causing thread interleaving anyway.

### Computer Engineering Justification
Instantiating an explicit Mutual Exclusion object (`threading.Lock`) acts as a strict hardware-level synchronization primitive. Wrapping the read-modify-write cycle into a `with self.lock:` statement guarantees that only a single sensor polling thread can safely mutate the system registry maps at any given clock cycle, eliminating state updates corruption.