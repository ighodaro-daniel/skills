# Solidity Linting Rules Reference

Detailed rules behind each linting pass. Read specific sections as needed while working.

## Table of Contents

1. [Imports](#imports)
2. [Unused Code](#unused-code)
3. [Naming](#naming)
4. [Visibility and Types](#visibility-and-types)
5. [Custom Errors](#custom-errors)
6. [Calldata vs Memory](#calldata-vs-memory)
7. [Magic Numbers](#magic-numbers)
8. [Declaration Ordering](#declaration-ordering)
9. [NatSpec](#natspec)
10. [forge fmt Rules](#forge-fmt-rules)

---

## Imports

### Named Imports (required)

Always use named imports. Bare imports pollute the namespace and make dependencies opaque.

```solidity
// Bad
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import * from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

// Good
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {IERC20, IERC20Metadata} from "@openzeppelin/contracts/interfaces/IERC20Metadata.sol";

// Aliased import (valid when avoiding name collision)
import {ERC20 as BaseERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

### Unused Imports

Remove any import whose symbol is never referenced in the file body. The Solidity compiler does NOT warn about these.

To detect: scan the file for every identifier from the import. If none appear after the import statements, remove the line.

---

## Unused Code

The Solidity compiler warns about unused local variables and function parameters, but NOT about:
- Unused state variables
- Unused events
- Unused custom errors
- Unused modifiers
- Unused private functions

### Unused Local Variables

```solidity
// Bad - result is assigned but never read
uint256 result = computeReward();
emit RewardProcessed(amount);

// Fix: remove the assignment or use the variable
```

### Unused Function Parameters

```solidity
// Bad - recipient never used in body
function process(uint256 amount, address recipient) external {
    _balances[msg.sender] -= amount;
}

// Good - make parameter unnamed when the signature must match an interface
function process(uint256 amount, address) external {
    _balances[msg.sender] -= amount;
}

// Also acceptable - underscore prefix signals intentionally unused
function process(uint256 amount, address _recipient) external {
    _balances[msg.sender] -= amount;
}
```

### Unused State Variables

Detect variables that are declared but never read or written to after removal of old logic. Be careful with upgradeable contracts - variables may be retained for storage layout. Flag these rather than removing silently.

### Unused Events, Errors, Modifiers

If a `emit`, `revert`, or modifier application never appears in the codebase, the definition can be removed.

---

## Naming

### Quick Reference Table

| Element | Convention | Example |
|---|---|---|
| Contract, Library | `PascalCase` | `TokenVault`, `SafeERC20` |
| Interface | `PascalCase` with `I` prefix | `IVault`, `IERC20` |
| Struct, Enum | `PascalCase` | `UserInfo`, `TokenStatus` |
| Event | `PascalCase` | `Transfer`, `OwnershipTransferred` |
| Custom Error | `PascalCase` | `InsufficientBalance`, `NotAuthorized` |
| Function (external/public) | `camelCase` | `getBalance`, `transferOwnership` |
| Function (internal/private) | `_camelCase` | `_computeReward`, `_validateInput` |
| Modifier | `camelCase` | `onlyOwner`, `nonReentrant` |
| State variable (public) | `camelCase` | `totalSupply`, `balanceOf` |
| State variable (internal/private) | `_camelCase` | `_owner`, `_totalReserves` |
| Local variable, parameter | `camelCase` | `totalAmount`, `tokenId` |
| Constant | `UPPER_CASE_WITH_UNDERSCORES` | `MAX_SUPPLY`, `BASIS_POINTS` |
| Immutable | `UPPER_CASE_WITH_UNDERSCORES` | `WETH`, `DEPLOYMENT_TIMESTAMP` |

### Interface Naming

Interfaces must start with `I`:

```solidity
// Bad
interface Vault { ... }
interface Token { ... }

// Good
interface IVault { ... }
interface IToken { ... }
```

### Private/Internal Underscore Prefix

```solidity
// State variables
uint256 private _totalReserves;
address internal _owner;

// Functions
function _computeReward(uint256 amount) internal returns (uint256) { ... }
function _validateInput(address user) private view { ... }
```

Public and external functions and state variables do NOT use underscores.

---

## Visibility and Types

### Explicit Visibility (always required)

Every state variable and function must have explicit visibility. Never rely on defaults.

```solidity
// Bad - state variable defaults to internal (not private, which surprises people)
uint256 _balance;

// Good
uint256 private _balance;

// Bad - function visibility unspecified
function _helper() { ... }

// Good
function _helper() internal { ... }
```

Prefer `private` over `internal` unless you specifically need derived contracts to access the member.

### Explicit Integer Types

Always use the full form:

```solidity
// Bad
uint x;
int y;

// Good
uint256 x;
int256 y;
```

`forge fmt` with default settings (`int_types = "long"`) auto-corrects these.

---

## Custom Errors

Since Solidity 0.8.4, custom errors are preferred over `require` strings. They are cheaper to deploy and revert, and provide structured machine-readable data.

### Conversion Pattern

```solidity
// Before
require(balance >= amount, "ERC20: insufficient balance");
require(msg.sender == owner, "Not authorized");

// After - define errors at the top of the contract (in the Errors section)
error InsufficientBalance(uint256 available, uint256 required);
error NotAuthorized(address caller);

// In function body
if (balance < amount) revert InsufficientBalance(balance, amount);
if (msg.sender != owner) revert NotAuthorized(msg.sender);
```

### Error Parameters

Add parameters when they help diagnose the problem. Omit them when the condition is self-documenting:

```solidity
// Parameters add value
error InsufficientBalance(uint256 available, uint256 required);

// Parameterless is fine when obvious
error Paused();
error ZeroAddress();
error Unauthorized();
```

### Naming

Custom errors use `PascalCase`. Describe the condition, not the action:

```solidity
// Good
error InsufficientBalance(...);
error DeadlineExceeded(uint256 deadline, uint256 current);
error InvalidSlippage(uint256 actual, uint256 max);

// Avoid
error Error1();
error FailedTransfer();  // vague
```

---

## Calldata vs Memory

For `external` functions only: reference-type parameters that are not modified inside the function body should use `calldata`.

```solidity
// Bad - copies data to memory unnecessarily
function processNames(string[] memory names) external {
    for (uint256 i = 0; i < names.length; i++) {
        emit NameLogged(names[i]);
    }
}

// Good - reads directly from calldata
function processNames(string[] calldata names) external {
    for (uint256 i = 0; i < names.length; i++) {
        emit NameLogged(names[i]);
    }
}

// Keep memory when the param is modified
function normalize(bytes memory data) external returns (bytes memory) {
    data[0] = 0x00; // modifying - must be memory
    return data;
}
```

**Does NOT apply to `public` functions** - they can be called internally which requires `memory`.

---

## Magic Numbers

Unexplained numeric literals reduce readability and make auditing harder. Extract them to named constants.

```solidity
// Bad
require(fee <= 1000, ...);
if (block.timestamp > lastAction + 604800) { ... }
uint256 penalty = amount * 250 / 10000;

// Good
uint256 private constant MAX_FEE_BPS = 1_000;    // 10%
uint256 private constant LOCK_PERIOD = 7 days;
uint256 private constant PENALTY_BPS = 250;       // 2.5%
uint256 private constant BASIS_POINTS = 10_000;

require(fee <= MAX_FEE_BPS, ...);
if (block.timestamp > lastAction + LOCK_PERIOD) { ... }
uint256 penalty = amount * PENALTY_BPS / BASIS_POINTS;
```

### Solidity Built-in Units

Use these instead of raw numbers:

```solidity
// Time
1 seconds, 1 minutes, 1 hours, 1 days, 1 weeks

// Ether
1 wei, 1 gwei, 1 ether

// Examples
uint256 private constant LOCK_PERIOD = 7 days;      // not 604800
uint256 private constant MIN_DEPOSIT = 0.01 ether;  // not 10000000000000000
```

### When NOT to Extract

Skip self-evident literals: `0`, `1`, simple loop indices, bit shifts where the amount is named in context, arithmetic involving `2**X` where X is clear.

---

## Declaration Ordering

Within a contract, follow this order:

```solidity
contract Example {
    // 1. using statements
    using SafeERC20 for IERC20;

    // 2. Constants
    uint256 private constant MAX_SUPPLY = 1_000_000e18;

    // 3. Immutables
    address public immutable WETH;

    // 4. State variables
    uint256 private _totalSupply;
    mapping(address => uint256) private _balances;

    // 5. Events
    event Transfer(address indexed from, address indexed to, uint256 value);

    // 6. Custom errors
    error InsufficientBalance(uint256 available, uint256 required);

    // 7. Modifiers
    modifier onlyOwner() { _; }

    // 8. Constructor
    constructor(address _weth) { WETH = _weth; }

    // 9. receive / fallback (if any)
    receive() external payable { ... }

    // 10. External functions
    function deposit(uint256 amount) external { ... }

    // 11. Public functions
    function totalSupply() public view returns (uint256) { ... }

    // 12. Internal functions
    function _update(address from, address to, uint256 value) internal { ... }

    // 13. Private functions
    function _computeFee(uint256 amount) private view returns (uint256) { ... }
}
```

Within each visibility group, prefer: payable functions first, then state-changing, then `view`, then `pure`.

---

## NatSpec

### Tag Reference

| Tag | Applies to | Notes |
|---|---|---|
| `@title` | contract, interface, library | One-line name |
| `@author` | contract, interface, library | Optional |
| `@notice` | contract, function, public state var, event, error | User-facing description |
| `@dev` | anything | Developer notes, impl details |
| `@param <name>` | function, event, error | One per parameter, name must match exactly |
| `@return` | function, public state var | One per return value |
| `@inheritdoc <Contract>` | function, public state var | Copies all tags from base; do not combine with other tags |
| `@custom:<tag>` | anything | Custom metadata |

### Syntax

```solidity
// Single tag - use triple-slash
/// @notice Returns the token balance of an account.

// Multiple tags - use block comment
/**
 * @notice Transfers tokens to a recipient.
 * @dev Emits a {Transfer} event. Reverts if balance is insufficient.
 * @param to The recipient address.
 * @param amount The number of tokens to transfer.
 * @return success Whether the transfer succeeded.
 */
function transfer(address to, uint256 amount) external returns (bool success);
```

### What Requires NatSpec

**External/public functions** - full NatSpec required:
```solidity
/// @notice Deposits tokens into the vault.
/// @param amount The number of tokens to deposit.
/// @return shares The number of vault shares minted.
function deposit(uint256 amount) external returns (uint256 shares) { ... }
```

**Contracts/interfaces** - title and notice required:
```solidity
/// @title Token Vault
/// @notice A yield-bearing vault that accepts ERC20 deposits.
contract Vault { ... }
```

**Internal/private functions** - `@dev` when non-obvious:
```solidity
/// @dev Assumes caller has already validated that `amount > 0`.
function _computeShares(uint256 amount) internal view returns (uint256) { ... }
```

**Events** - notice and params:
```solidity
/// @notice Emitted when tokens are deposited.
/// @param depositor The address that deposited.
/// @param amount The amount deposited.
event Deposited(address indexed depositor, uint256 amount);
```

**Custom errors** - notice:
```solidity
/// @notice Reverted when the caller's balance is too low.
error InsufficientBalance(uint256 available, uint256 required);
```

**Public state variables** - notice:
```solidity
/// @notice The total number of tokens in circulation.
uint256 public totalSupply;
```

### `@inheritdoc`

Use in implementations to avoid repeating interface documentation:

```solidity
interface IVault {
    /// @notice Deposits tokens and mints shares.
    /// @param amount The deposit amount.
    /// @return shares The minted shares.
    function deposit(uint256 amount) external returns (uint256 shares);
}

contract Vault is IVault {
    /// @inheritdoc IVault
    function deposit(uint256 amount) external returns (uint256 shares) { ... }
}
```

Do NOT mix `@inheritdoc` with other tags on the same function.

---

## forge fmt Rules

Default `forge fmt` settings (from `foundry.toml` defaults):

| Setting | Default | Effect |
|---|---|---|
| `line_length` | 120 | Wrap lines exceeding this length |
| `tab_width` | 4 | 4 spaces per indent level (always spaces, never tabs) |
| `bracket_spacing` | false | No spaces inside `{}`: `{a: 1}` not `{ a: 1 }` |
| `int_types` | `"long"` | `uint` -> `uint256`, `int` -> `int256` |
| `quote_style` | `"double"` | All strings use `"double quotes"` |
| `number_underscore` | `"preserve"` | Numeric underscore separators left as-is |
| `sort_imports` | false | Import order preserved |

**Always enforced (not configurable):**
- Trailing newline at end of file
- No trailing whitespace
- Opening brace on same line as declaration
- Consistent blank lines between top-level definitions
- Import formatting: `import {A, B} from "path"` on one line if it fits
