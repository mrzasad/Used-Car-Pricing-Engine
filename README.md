# Used Car Valuation Engine

A .NET class library and valuation algorithm that determines the fair market value of a used vehicle by sequentially applying adjustments for:

- Vehicle age
- Mileage
- Ownership history
- Collision history
- Brand reliability

The engine processes adjustments in order, with each step modifying the running total inherited from the previous step.

---

# ⚙️ Calculation Pipeline

```text
[Purchase Price]
       │
       ▼
1. AGE ──────────► -0.5% per month (Max 120 months)
       │
       ▼
2. MILEAGE ──────► -0.2% per 1,000 miles (Max 150k miles)
       │
       ▼
3. OWNERS > 2 ───► -25% penalty (Skip if ≤ 2 owners)
       │
       ▼
4. COLLISIONS ───► -2% per incident (Max 5 collisions)
       │
       ▼
5. RELIABILITY ──► Toyota: +5% | Ford: -$500
       │
       ▼
6. OWNERS = 0 ───► +10% bonus (Applied after Step 5)
       │
       ▼
[PROFIT CAP] ────► Final value capped at 90% of purchase price
```

---

# 📊 Business Rules

## 1. Age Depreciation

- **Rate:** Lose `0.5%` of current value per month
- **Limit:** Maximum `120 months` (10 years)

### Formula

```text
current -= current * (months * 0.005)
```

---

## 2. Mileage Depreciation

- **Rate:** Lose `0.2%` of current value per `1,000` miles
- **Limit:** Maximum `150,000` miles
- **Note:** Remaining miles under `1,000` are ignored

### Formula

```text
current -= current * (thousands * 0.002)
```

---

## 3. Ownership Impact

### High Ownership Penalty

If:

```text
owners > 2
```

Apply:

```text
-25%
```

### Zero Ownership Bonus

If:

```text
owners == 0
```

Apply:

```text
+10%
```

This bonus is applied **after manufacturer reliability adjustments**.

---

## 4. Collision Penalty

- **Rate:** `-2%` per collision
- **Limit:** Maximum `5` collisions

### Maximum Reduction

```text
10%
```

---

## 5. Manufacturer Reliability

### Toyota

```text
+5% value increase
```

### Ford

```text
-$500 flat deduction
```

---

## 6. Profitability Protection

The final valuation can never exceed:

```text
90% of the original purchase price
```

---

# 🚀 Quick Start

## Prerequisites

- .NET 8.0 SDK or higher

---

# 📦 Installation

```bash
git clone https://github.com/<your-username>/used-car-valuation.git

cd used-car-valuation

dotnet restore
```

---

# 🧩 Data Model

```csharp
public record CarInput(
    decimal PurchasePrice,
    int AgeInMonths,
    int Mileage,
    int PreviousOwners,
    int Collisions,
    string Make
);
```

---

# 🧠 Reference Implementation

```csharp
public class ValuationEngine
{
    public decimal CalculateValue(CarInput car)
    {
        decimal current = car.PurchasePrice;

        // 1. Age
        int ageMonths = Math.Min(car.AgeInMonths, 120);
        current -= current * (ageMonths * 0.005m);

        // 2. Mileage
        int eligibleMiles = Math.Min(car.Mileage, 150000);
        int thousands = eligibleMiles / 1000;
        current -= current * (thousands * 0.002m);

        // 3. Previous Owners (Negative Impact)
        if (car.PreviousOwners > 2)
        {
            current *= 0.75m;
        }

        // 4. Collisions
        int limitedCollisions = Math.Min(car.Collisions, 5);
        current -= current * (limitedCollisions * 0.02m);

        // 5. Reliability
        if (car.Make.Equals("Toyota", StringComparison.OrdinalIgnoreCase))
        {
            current *= 1.05m;
        }
        else if (car.Make.Equals("Ford", StringComparison.OrdinalIgnoreCase))
        {
            current -= 500m;
        }

        // 6. Previous Owners (Positive Impact)
        if (car.PreviousOwners == 0)
        {
            current *= 1.10m;
        }

        // Profitability Cap
        decimal maxPrice = car.PurchasePrice * 0.90m;

        return Math.Min(current, maxPrice);
    }
}
```

---

# ✅ Example Usage

```csharp
var car = new CarInput(
    PurchasePrice: 30000m,
    AgeInMonths: 36,
    Mileage: 45000,
    PreviousOwners: 1,
    Collisions: 2,
    Make: "Toyota"
);

var engine = new ValuationEngine();

decimal estimatedValue = engine.CalculateValue(car);

Console.WriteLine($"Estimated Value: {estimatedValue:C}");
```

---

# 🧪 Running Tests

```bash
dotnet test
```

---

# 📁 Suggested Project Structure

```text
used-car-valuation/
│
├── src/
│   └── UsedCarValuation/
│       ├── CarInput.cs
│       ├── ValuationEngine.cs
│       └── BusinessRules/
│
├── tests/
│   └── UsedCarValuation.Tests/
│
├── README.md
└── UsedCarValuation.sln
```

---

# 🔍 Example Scenarios

| Scenario | Result |
|---|---|
| New Toyota with no owners | Gains reliability + ownership bonus |
| Older high-mileage vehicle | Heavy depreciation |
| Ford with multiple owners | Ownership penalty + flat deduction |
| Collision-heavy vehicle | Collision reduction capped at 10% |

---

# 📈 Design Goals

- Deterministic valuation logic
- Sequential rule processing
- Easily testable
- Extensible business rules
- Simple .NET integration

---

# 🛠️ Future Improvements

- ASP.NET Core Web API
- Rule engine abstraction
- Configuration-based valuation rules
- Database persistence
- VIN decoding integration
- Market trend adjustments
- ML-assisted pricing models

---

# 📄 License

MIT License

---

# 🤝 Contributing

Pull requests and improvements are welcome.

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Open a pull request
