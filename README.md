from qiskit import QuantumCircuit
from qiskit.primitives import StatevectorSampler
import numpy as np

n = 12

alice_bits = np.random.randint(2, size=n)
alice_basis = np.random.randint(2, size=n)

eve_basis = np.random.randint(2, size=n)   # Eve chooses random basis
bob_basis = np.random.randint(2, size=n)

key = []
matched_indices = []
errors = 0

sampler = StatevectorSampler()

for i in range(n):
    qc = QuantumCircuit(1, 1)
    
    if alice_bits[i] == 1:
        qc.x(0)
    if alice_basis[i] == 1:
        qc.h(0)
    
    
    if eve_basis[i] == 1:
        qc.h(0)
    
    qc.measure(0, 0)
    
    result_eve = sampler.run([qc], shots=1).result()
    counts_eve = result_eve[0].data[qc.cregs[0].name].get_counts()
    eve_meas = int(next(iter(counts_eve)))
    
    
    qc = QuantumCircuit(1, 1)
    
    if eve_meas == 1:
        qc.x(0)
    if eve_basis[i] == 1:
        qc.h(0)
    
   
    if bob_basis[i] == 1:
        qc.h(0)
    
    qc.measure(0, 0)
    
    result_bob = sampler.run([qc], shots=1).result()
    counts_bob = result_bob[0].data[qc.cregs[0].name].get_counts()
    bob_meas = int(next(iter(counts_bob)))
    
    
    if alice_basis[i] == bob_basis[i]:
        matched_indices.append(i)
        key.append(bob_meas)
        
        # Check error
        if bob_meas != alice_bits[i]:
            errors += 1

print("Alice bits:      ", alice_bits)
print("Alice basis:     ", alice_basis)
print("Eve basis:       ", eve_basis)
print("Bob basis:       ", bob_basis)
print("Matched indices: ", matched_indices)
print("Shared key:      ", key)

if len(matched_indices) > 0:
    error_rate = errors / len(matched_indices)
else:
    error_rate = 0

print("Error rate:      ", error_rate)


from qiskit import QuantumCircuit
from qiskit.primitives import StatevectorSampler

def grover_2qubit(target):
    qc = QuantumCircuit(2, 2)

    # Step 1: Superposition
    qc.h(0)
    qc.h(1)

    # --- ORACLE (depends on target) ---
    # Map target → |11⟩
    if target[0] == '0':
        qc.x(0)
    if target[1] == '0':
        qc.x(1)

    qc.cz(0, 1)   # mark |11⟩

    # Undo mapping
    if target[0] == '0':
        qc.x(0)
    if target[1] == '0':
        qc.x(1)

    # --- DIFFUSION ---
    qc.h(0)
    qc.h(1)

    qc.x(0)
    qc.x(1)

    qc.cz(0, 1)

    qc.x(0)
    qc.x(1)

    qc.h(0)
    qc.h(1)

    # Measurement
    qc.measure([0,1], [0,1])

    return qc


# 🔹 Choose target here
target = "10"

qc = grover_2qubit(target)

sampler = StatevectorSampler()
result = sampler.run([qc], shots=1024).result()

counts = result[0].data[qc.cregs[0].name].get_counts()
print("Target:", target)
print("Output:", counts)

print(qc.draw())
