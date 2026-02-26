# 🚀 Practical Deployment & Reproduction Guide
This section provides detailed instructions for configuring, deploying, and reproducing the Gaia semi-constrained navigation system. The implementation strictly follows the two-party semi-honest model (Navigation Server and Proxy) as described in the paper.

## 💻 1. System Requirements & Environment Setup
Gaia relies on heavy cryptographic operations (CKKS homomorphic encryption, Garbled Circuits and Oblivious Transfer). Please ensure your environment meets the following requirements:

### 1.1 Hardware Requirements
- **Memory (RAM):** Minimum 16 GB required. The Encrypted Distance Matrix (EDM) for the 2,200-node graph requires significant memory. Our pom.xml is explicitly configured with JVM arguments `-Xms8g -Xmx16g` to prevent `OutOfMemoryError`.
- **CPU:** Multi-core processor (Intel/AMD) to support efficient polynomial operations.

### 1.2 Software Dependencies
- **JDK:** Java 11 or higher.
- **Build Tool:** Maven 3.6+.
- **Cryptographic Libraries (Included in project):**
  - SCAPI (Secure Computation API): Used for executing Garbled Circuits during ciphertext comparisons. Note: The project uses `ScapiWin-V2-3-0.jar`, ensuring compatibility with the Windows native cryptographic environment.
  - BouncyCastle: Configured via `bcprov-jdk16` for foundational crypto functions.

## 🏗️ 2. System Architecture & Component Deployment
Gaia is designed as a distributed two-party system. In this repository, the Navigation Server (NS) and the Secure Proxy are implemented to communicate via local socket connections to simulate the real-world physical separation of keys.

### 📂 2.1 Offline Graph Preprocessing
Before handling online queries, the system must generate the shortest path distance matrices and static Deviation Stops (DSt) for obstacle avoidance.
- **Target Class:** `Floyd_Warshall_1.java`
- **Action:** Run this class to parse the initial road network (e.g., the 2,200-node OpenStreetMap dataset). It computes the static topologies and logical connections necessary for routing.

### 🔑 2.2 Secure Proxy Initialization (Key Holder)
The Proxy holds the Secret Key (SK) and participates strictly in decryption and Garbled Circuit comparisons. It does NOT hold the map data.
- **Target Class:** `CT.java` (Cryptographic Tools)
- **Action:** The Proxy initializes the CKKS context (e.g., polynomial degree $N=8192$) and listens for secure comparison requests from the NS.
- **Implementation Detail:** The system establishes Socket connections (e.g., listening on a specific port like 12345 via localhost for testing) to perform the `CT_CMP2` (Ciphertext Comparison) protocol using SCAPI's garbled circuits.

### 🗺️ 2.3 Navigation Server Execution (Data Holder)
The NS holds the Encrypted Distance Matrix (EDM) and the CKKS Evaluation Keys (RelinKeys). It executes the core homomorphic navigation logic.
- **Target Class:** `GaiaNavigationTools.java`
- **Action:** The NS runs the geometric routing algorithms (SecureSkimming, NLow, NMid, NHigh) entirely over ciphertexts. When it needs to compare two encrypted distances, it interacts with the Proxy over the configured socket.

## 🧪 3. Reproducing Experimental Results
To reproduce the performance evaluation metrics (Time Breakdowns, Path Dilation Rate, Hybrid Path Similarity) presented in our paper, we provide an automated test suite.

### 3.1 Configure Test Parameters
Open `Test3.java` to adjust the evaluation scales. You can customize the following arrays:
```java
// Example configuration in Test3.java
private static final int[] WAYPOINT_COUNTS = { 5 }; // Number of stops (k)
private static final int[] CONSTRAINT_PAIRS = { 1, 3, 5 }; // Partial orders
private static final double[] BLOCKAGE_RATIOS = { 0.05, 0.15 }; // Edge blockages
```

### 3.2 Execute the Test Suite
Use Maven to compile and run the main test class. Ensure the memory arguments are applied:
```bash
mvn clean test-compile
MAVEN_OPTS="-Xms8g -Xmx16g" mvn exec:java -Dexec.mainClass="Test3" -Dexec.classpathScope=test
```

### 3.3 Output & Analysis
Upon completion, the system will output the experimental results into a CSV file (e.g., `gaia_performance400.csv` or `gaia_performance2200.csv` depending on the dataset loaded). This file includes:
- Encryption/Decryption Time
- Token Generation Time
- Query Execution Time
- LenSim (Length Similarity against baseline TSP)
- HPS (Hybrid Path Similarity)
- ……

### 🛡️ 4. Security Note
The current deployment strictly follows the semi-honest adversary model. 
The Proxy and NS are non-colluding entities. 
All geographic features, visit orders, and user queries remain encrypted throughout the entire navigation process.
