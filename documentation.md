# documentation.md

## Class Explanations (Phase I)

### **Classes**

* **User (Abstract):** The base class for all actors (Students, Staff) containing shared identity attributes like `userId`, `phoneNumber`, and `idProofUrl`. It defines core methods for `register()`, `login()`, and `viewHistory()`.


* **Student:** A subclass of `User` that adds a unique `rollNumber`. It includes logic to `getFeeAccountDetails()` for transaction processing.


* **Staff:** A subclass of `User` that includes an `employeeId`. It contains logic to `getSalaryAccountDetails()` to facilitate payroll deductions.


* **Wallet:** Handles the user's `balance` and an `autoDeductEnabled` flag. It acts as the primary interface for `addMoney()` and `deductMoney()` operations.


* **PaymentMethod (Abstract):** Defines the `processPayment()` contract that all concrete strategies must implement to decouple the Wallet from specific providers.


* **UPIPayment:** A concrete strategy for external UPI transfers that handles `initiateTransaction()` and `verifyWithBank()` logic.


* **SalaryDeduction:** A concrete strategy restricted to `Staff` that implements `verifyEmployeeAccount()` and `scheduleDeduction()` via the payroll system.


* **FeeAddition:** A concrete strategy restricted to `Students` that handles `verifyStudentAccount()` and `addToFeeStructure()`.


* **Vehicle (Abstract):** The base IoT hardware class tracking `batteryLevel`, `gpsLocation`, and `VehicleStatus`. It manages hardware states through `lock()`, `unlock()`, and `updateStatus()`.


* **Bicycle / Moped / SmartBike:** Concrete subclasses representing hardware types with specific attributes such as `gearCount` (Bicycle), `fuelLevel` (Moped), or `electricRange` (SmartBike).


* **ParkingLot:** Manages designated zones by tracking `capacity` and `currentCount`. It uses `isBikeInside()` to validate location via GPS coordinates.


* **Trip:** Manages the ride lifecycle, storing `startTime`, `endTime`, `distanceCovered`, and `totalFare`. It coordinates with `RateCard` for fare computation.


* **RateCard:** Encapsulates pricing rules, including `baseRate`, `ratePer100m`, and the `dailyFine` logic.


* **Transaction:** A permanent audit record of financial events, including `amount`, `timestamp`, and `type` (e.g., recharge or trip payment).


* **Feedback:** Stores the post-trip `rating` (int) and `comment` (String) provided by the user.


* **BikeRentalController:** A manager class that orchestrates the system by initiating trips, completing transactions, and managing wallet recharges.


### **Inheritance**

* **User Hierarchy:** `Student` and `Staff` inherit from the abstract `User` class to share core authentication and profile logic.


* **Payment Strategy:** `UPIPayment`, `SalaryDeduction`, and `FeeAddition` extend `PaymentMethod`, allowing for polymorphic processing of different payment types.


* **Vehicle Hierarchy:** `Bicycle`, `Moped`, and `SmartBike` extend the base `Vehicle` class to support diverse hardware while maintaining a unified interface for the controller.


### **Relationships & Cardinalities**

* **User — Wallet (1:1 Composition):** Each User owns exactly one Wallet instance for all financial activity.


* **User — Trip (1:0..* Association):** A single User can take multiple Trips over time.


* **Trip — Vehicle (0..*:1 Association):** Each individual Trip is linked to exactly one Vehicle instance.


* **Vehicle — ParkingLot (0..*:1 Aggregation):** Vehicles are logically docked at a ParkingLot, which aggregates multiple vehicles.


* **Trip — Transaction (1:1 Composition):** Every completed Trip generates exactly one Transaction record.


* **Trip — Feedback (1:0..1 Composition):** A Trip can optionally receive one set of Feedback from the user.


* **Transaction — PaymentMethod (1:1 Association):** Each transaction is processed via exactly one selected PaymentMethod strategy.

## Assumptions

1. **Persistent Connectivity:** All vehicles and user mobile applications maintain continuous internet access to sync status and GPS data.


2. **GPS Precision:** Phone and vehicle GPS sensors are accurate enough to validate geofenced parking within `ParkingLot` boundaries.


3. **Third-Party API Availability:** University payroll and fee management systems provide stable APIs for the `SalaryDeduction` and `FeeAddition` classes.


4. **Hardware Protocols:** Different vehicle types (Bicycle, Moped) utilize a standardized IoT protocol for remote locking and unlocking.



## Additional Features

1. **Payment Strategy Pattern:** Implemented a flexible Strategy Pattern for payments, allowing new methods (like Credit Cards) to be added without modifying the `Wallet` class.


2. **Pre-Trip Resource Validation:** The system checks both the vehicle's `batteryLevel` and the user's `minimumRequired` wallet balance before allowing a trip to start.


3. **Geofenced Location Validation:** The `ParkingLot` class uses `isBikeInside()` to strictly enforce that trips only end within designated zones via GPS validation.


4. **Centralized Control Hub:** Included a `BikeRentalController` to manage complex interactions between users, hardware, and financial records.


## Phase III: Comparative Analysis (Human vs. LLM Review)

### 1. Structural Comparison (Manual vs. LLM)

| Feature | Manual Design (Phase I) | LLM Design (Phase II) | Analysis |
| :--- | :--- | :--- | :--- |
| **Vehicle Hierarchy** |**Polymorphic**: Defines `Bicycle`, `Moped`, and `SmartBike` as distinct subclasses with unique attributes like `gearCount` and `fuelLevel`[cite: 100]. | **Singular**: Only implemented a single `SmartBike` class. | The LLM failed to support the requirement for multiple vehicle types (bike, bicycle, or moped). |
| **Payment Logic** |**Advanced Strategies**: Includes specific methods like `verifyWithBank()` [cite: 99] [cite_start]and `scheduleDeduction()` [cite: 106] within the strategy pattern. | **Basic**: Uses a simple association between `Wallet` and `Payment` records. | The manual design correctly captures the IIIT-H specific workflows for Salary and Fees. |
| **Parking System** |**Geofenced**: The `ParkingLot` validates presence using GPS coordinates via the `isBikeInside()` method[cite: 100, 114]. | **Hardware-based**: Relies on a `DockingStation` physical lock model. | Manual geofencing is more scalable for a campus setting and reduces infrastructure costs. |
| **Pre-Ride Checks** |**Resource-Aware**: Explicitly validates both `batteryLevel` and `checkBalance()` before triggering the `unlock()` command[cite: 110, 111]. | **Transactional**: Assumed unlocking occurs immediately upon scanning without state validation. | The manual design prevents "stranded" trips by verifying resources before the trip starts. |

### 2. IIIT-H Specific Constraint Analysis

* **Constraint: "Users (staff, student, teacher)"**
  * **LLM Performance:** **High.** The LLM correctly identified these types, though it added `Faculty` as a separate class while the Manual design grouped teachers under `Staff`.


* **Constraint: "Deduct from salary/fees"**
  * **LLM Performance:** **High.** Both designs correctly implemented `SalaryDeduction` and `FeeAddition`.


* **Constraint: "Bike, bicycle or moped"**
  * **LLM Performance:** **Low.** The LLM only provided a generic `SmartBike`, missing the specific attributes for human-powered or fuel-based vehicles.


* **Constraint: "Designated parking lots"**
  * **LLM Performance:** **Medium.** The LLM assumed physical docks, whereas the manual design correctly interprets "parking lot" to include virtual zones via geofencing.


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
