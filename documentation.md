# documentation.md

## Class Explanations (Phase I)

### **Classes**

* **User (Abstract):** The base class for all actors (Students, Staff) containing shared attributes like ID proof and methods for login and scanning QR codes.
* **Student:** Subclass of User that includes `rollNumber` and logic to authorize `FeeAddition` payments.
* **Staff:** Subclass of User that includes `employeeId` and logic to authorize `SalaryDeduction` payments.
* **Wallet:** Handles the user's balance and `autoDeduct` functionality, acting as the client that utilizes various `PaymentMethod` strategies.
* **PaymentMethod (Abstract):** Defines the contract `processPayment` that all concrete payment types (UPI, Salary, Fee) must implement.
* **UPIPayment:** A concrete payment method that integrates with external UPI interfaces to transfer funds to the Wallet.
* **SalaryDeduction:** A concrete payment method exclusive to Staff that communicates with the payroll system.
* **FeeAddition:** A concrete payment method exclusive to Students that adds charges to their semester fee account.
* **Vehicle (Abstract):** The base hardware class containing shared state (GPS, battery) and control methods (lock/unlock) for all vehicle types.
* **Bicycle / Moped / SmartBike:** Concrete subclasses of Vehicle representing specific hardware types with unique attributes like `gearCount` or `fuelLevel`.
* **ParkingLot:** Manages the physical or geofenced zones where vehicles can be legally parked and tracks current capacity.
* **Trip:** Manages the lifecycle of a ride, linking a User to a specific Vehicle instance and calculating the final fare.
* **RateCard:** Encapsulates the pricing rules (base rate, distance increments, fines) to decouple logic from the Trip class.
* **Transaction:** A permanent record of any successful payment or wallet top-up for auditing purposes.
* **Feedback:** Stores the rating and comments provided by a user after a trip concludes.

### **Inheritance**

* **User Hierarchy:** `Student` and `Staff` inherit from `User` to share identity logic while enabling distinct payment privileges.
* **Payment Hierarchy:** `UPIPayment`, `SalaryDeduction`, and `FeeAddition` inherit from `PaymentMethod` to implement specific processing logic (Polymorphism).
* **Vehicle Hierarchy:** `Bicycle`, `Moped`, and `SmartBike` inherit from `Vehicle` to share core IoT functionality while allowing for hardware-specific attributes.

### **Relationships & Cardinalities**

* **User — Wallet (1:1 Composition):** Each User has exactly one Wallet instance.
* **Wallet — PaymentMethod (1:1 Dependency):** The Wallet uses a transient `PaymentMethod` instance whenever money is added.
* **User — Trip (1:0..* Association):** A User can take multiple Trips.
* **Trip — Vehicle (0..*:1 Association):** A Trip uses exactly one Vehicle.
* **Vehicle — ParkingLot (0..*:1 Aggregation):** Vehicles are located at a ParkingLot but can exist independently (during a trip).
* **Trip — Transaction (1:1 Composition):** Every Trip results in one Transaction.
* **Trip — RateCard (Dependency):** Trip depends on RateCard for fare calculation.

## Assumptions

1. **Internet Connectivity:** All vehicles and user apps have persistent internet access to communicate with the central server.
2. **GPS Accuracy:** The GPS modules on vehicles and phones are accurate enough to strictly enforce geofenced parking.
3. **Payment API Integration:** The University's Payroll and Fee systems provide accessible APIs for the `SalaryDeduction` and `FeeAddition` classes.
4. **Hardware Standardization:** All vehicle subclasses (Moped, Bicycle) share the same locking/unlocking IoT protocol despite physical differences.

## Additional Features

1. **Payment Strategy Pattern:** Implemented a flexible Strategy Pattern for payments, allowing easy addition of new payment methods (e.g., Credit Card) without modifying the Wallet class.
2. **Vehicle Polymorphism:** Designed a Vehicle hierarchy to easily support future vehicle types (e.g., Electric Scooters) by extending the abstract base class.
3. **Geofenced Parking:** Implemented virtual docking stations using GPS coordinates in the `ParkingLot` class to reduce infrastructure costs.


## Phase III: Comparative Analysis (Human vs. LLM Review)

### 1. Structural Comparison (Manual vs. LLM)

The following table contrasts the architectural choices made in the **Manual Design (Phase I)** against the **LLM-generated Design (Phase II)**.

| Feature | Manual Design (Phase I) | LLM Design (Phase II) | Analysis |
| --- | --- | --- | --- |
| **Vehicle Hierarchy** | **Polymorphic:** Abstract `Vehicle` class with subclasses `Bicycle`, `Moped`, `SmartBike`. | **Singular:** Single `SmartBike` class. | **Correction Needed:** The LLM failed to capture the requirement for multiple vehicle types ("bike, bicycle or moped"). The manual design correctly allows for different behaviors (fuel vs. battery) per vehicle type. |
| **Parking Logic** | **Geofenced:** `ParkingLot` class checks GPS coordinates (Virtual Docking). | **Hardware-based:** `ParkingLot` aggregates `DockingStation` objects. | **Critique:** The LLM assumed a physical docking station model. While valid, the Manual design's geofencing approach is more scalable for a campus setting and reduces infrastructure constraints. |
| **User Hierarchy** | **Simplified:** `Student` and `Staff`. | **Granular:** `Student`, `Staff`, and `Faculty`. | **Alignment:** Both designs correctly capture the need to separate users. The LLM added `Faculty` explicitly, which aligns with the "teacher" requirement, whereas the Manual design grouped teachers under `Staff`. |
| **Payment & Wallet** | **Strategy Pattern:** `Wallet` injects `PaymentMethod` dependencies directly. | **Relational:** `Wallet` links to `Payment` records, which link to `PaymentMethod`. | **Alignment:** Both designs correctly identified the specific IIIT-H payment constraints (`SalaryDeduction`, `FeeAddition`). The LLM's separation of a `Payment` transaction record from the method is a good database normalization practice. |
| **Pricing Logic** | **Unified:** Single `RateCard` class handling base rates and fines. | **Decoupled:** Separate `PricingPolicy` and `FinePolicy` classes. | **Critique:** The LLM's approach to separate Pricing and Fines is cleaner and adheres better to the Single Responsibility Principle, allowing fines to evolve independently of ride rates. |
| **Support System** | **Implicit:** Handled via `Feedback` class. | **Explicit:** Dedicated `SupportDocument` class linked to `User`. | **Alignment:** The LLM correctly picked up on the requirement for "Support information... documentation," which the Manual design treated more abstractly. |

### 2. IIIT-H Specific Constraint Analysis

* **Constraint: "Users (staff, student, teacher)"**
  * **LLM Performance:** **High.** The LLM correctly identified the three user types and created specific classes for them.


* **Constraint: "Deduct from salary/fees"**
  * **LLM Performance:** **High.** The LLM correctly implemented `SalaryDeduction` and `FeeAddition` classes inheriting from `PaymentMethod`.


* **Constraint: "Bike, bicycle or moped"**
  * **LLM Performance:** **Low.** The LLM ignored the variety of vehicles, implementing only a generic `SmartBike` class. This misses potential attributes like `fuelLevel` for mopeds vs `battery` for e-bikes.


* **Constraint: "Designated parking lots"**
  * **LLM Performance:** **Medium.** The LLM introduced `DockingStation`, implying a strict physical lock mechanism, whereas the problem description allowed for a broader interpretation of "parking lot" which is often solved via geofencing in modern apps.



### 3. Specific Corrections Required for LLM Output

To ensure the LLM's design fully meets the requirements defined in the problem statement, the following specific modifications are necessary:

1. **Refactor Vehicle Class:**
* **Current:** Single `SmartBike` class.
* **Correction:** Convert `SmartBike` into an abstract `Vehicle` class. Create subclasses `Bicycle` (human-powered), `Moped` (fuel-based), and `E-Bike` (battery-based) to accommodate the different vehicle types mentioned.


2. **Simplify Parking Architecture:**
* **Current:** `ParkingLot` -> `DockingStation` -> `Bike`.
* **Correction:** Remove `DockingStation` unless physical docks are explicitly mandatory. Allow `ParkingLot` to aggregate `Vehicle` directly based on GPS presence to support "virtual" parking zones.


3. **Enhance Trip Logic:**
* **Current:** `Trip` connects User and Bike.
* **Correction:** Ensure `Trip` explicitly links to `ParkingLot` for both `startLocation` and `endLocation` to validate that rides only begin/end at designated spots, as per the "Trips can only be started and ended at designated parking lots" requirement.


4. **Integrate Payment with User Types:**
* **Current:** `PaymentMethod` subclasses exist but aren't explicitly restricted to user types in the diagram.
* **Correction:** Add constraints or associations ensuring `SalaryDeduction` is *only* accessible by `Staff/Faculty` and `FeeAddition` is *only* accessible by `Student`.


## Contributions

**Amritha Vinod (2023101075):**

**Ananya Vishal Halgatti (2023101129):**

**Janya Gupta (2022102033):**

**Kriti Gupta (2023101123):**

**Priyanshi Gupta (2023101068):**
