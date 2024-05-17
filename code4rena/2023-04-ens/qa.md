`Dear ENS team, as we have gone through each contract within the scope, we have noticed good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing. We understand that every team has a different level of good practices, but we believe that at least 90% of the recommendations in the following report should be applied for better gas efficiency, readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future projects.`


### Issues
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| S | Suggestions | Suggestion Details |

| Total Found Issues | 18 |
|:--:|:--:|

### Low Risk
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | INCONSISTENT SOLIDITY PRAGMA | 4 |
| [L-02] | LACK OF ADDRESS(0) CHECKS IN THE CONSTRUCTOR | 1 |
| [L-03] | ASSEMBLY CODES SPECIFIC - SHOULD HAVE COMMENTS | 15 |
| [L-04] | MISSING DOCSTRINGS | 11 |
| [L-05] | FUNCTION OVERLOADING | 8 |
| [L-06] | WE SUGGEST USING THE OPENZEPPELIN SAFECAST LIBRARY | 2 |

| Total Low Issues | 6 | Total Instances | 41 |
|:--:|:--:|:--:|--:|

### Non-Critical
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | MISSING VISIBILITY IN VARIABLES | 29 |
| [N-02] | USING WHILE FOR UNBOUNDED LOOPS ISN'T RECOMMENDED | 8 |
| [N-03] | SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE | 14 |
| [N-04] | TAKE ADVANTAGE OF CUSTOM ERROR'S RETURN VALUE PROPERTY | 3 |
| [N-05] | CONTRACT DOES NOT FOLLOW THE SOLIDITY STYLE GUIDE'S SUGGESTED LAYOUT ORDERING | 2 |
| [N-06] | MISSING SPDX LICENSE IDENTIFIER | 1 |
| [N-07] | MANDATORY CHECKS FOR EXTRA SAFETY IN THE SETTERS | 1 |
| [N-08] | ALSO VALUES IN ASSEMBLY CAN BE STORED IN CONSTANT VARIABLES | 5 |
| [N-09] | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS | 31 |

| Total Non-Critical Issues | 9 | Total Instances | 94 |
|:--:|:--:|:--:|--:|

### Refactor Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | FUNCTION NAMING SUGGESTIONS (Underscore Prefix for Non-external Functions and Variables) | 75 |
| [R-02] | SHORTHAND WAY TO WRITE IF/ELSE STATEMENT | 8 |

| Total Refactor Issues | 2 | Total Instances | 84 |
|:--:|:--:|:--:|--:|


### Suggestion Details 
| Count | Explanation | 
|:--:|:-------|
| [S-01] | WE SUGGEST TO USE NAMED PARAMETERS FOR MAPPING TYPE DECLARATIONS |  | 

| Total Suggestions | 1 |
|:--:|:--:|

# Detailed Findings

## [L-01] INCONSISTENT SOLIDITY PRAGMA
The source files have different solidity compiler ranges referenced. This leads to potential security flaws between deployed contracts depending on the compiler version chosen for any particular file. It also greatly increases the cost of maintenance as different compiler versions have different semantics and behavior.

### PROOF OF CONCEPT
```solidity
contracts/dnsregistrar/DNSRegistrar.sol
3: pragma solidity ^0.8.4;

contracts/dnsregistrar/RecordParser.sol
pragma solidity ^0.8.11;

contracts/utils/NameEncoder.sol
2: pragma solidity ^0.8.13;

contracts/wrapper/BytesUtils.sol
2: pragma solidity ~0.8.17;
```

### MITIGATION
We recommend to fix a definite compiler range that is consistent between contracts and upgrade any affected contracts to conform to the specified compiler.

## [L-02] LACK OF ADDRESS(0) CHECKS IN THE CONSTRUCTOR
Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly.

```solidity
contracts/dnsregistrar/DNSRegistrar.sol
55: constructor(
        address _previousRegistrar,
        address _resolver,
        DNSSEC _dnssec,
        PublicSuffixList _suffixes,
        ENS _ens
    ) {
        previousRegistrar = _previousRegistrar;
        resolver = _resolver;
        oracle = _dnssec;
        suffixes = _suffixes;
        emit NewPublicSuffixList(address(suffixes));
        ens = _ens;
    }
```
### MITIGATION
Consider adding zero-address checks in the discussed constructors:  

```solidity
require(newAddr != address(0));
```

## [L-03] ASSEMBLY CODES SPECIFIC - SHOULD HAVE COMMENTS
Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

### PROOF OF CONCEPT
```solidity
// @audit there 11 instance in BytesUtils.sol

contracts/dnssec-oracle/RRUtils.sol
386:  assembly {
            word := mload(add(add(data, 32), i))
      }

contracts/dnssec-oracle/algorithms/ModexpPrecompile.sol
23: assembly {
        success := staticcall(
            gas(),
            5,
            add(input, 32),
            mload(input),
            add(output, 32),
            mload(modulus)
        )
    }

contracts/dnssec-oracle/SHA1.sol
7:  assembly {
            // Get a safe scratch location
            let scratch := mload(0x40)

            // Get the data length, and point data at the first byte
            let len := mload(data)
            data := add(data, 32)

            // Find the length after padding
            let totallen := add(and(add(len, 1), 0xFFFFFFFFFFFFFFC0), 64)
            switch lt(sub(totallen, len), 9)
            case 1 {
                totallen := add(totallen, 64)
            }          
    }
```
Also in the following contract:
- [HexUtils.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/utils/HexUtils.sol#L17-L59)

### RECOMMENDATION
It is essential to clearly and comprehensively document all activities related to critical function `assembly` within the project. By doing so, a more complete and accurate understanding of what is being done is provided, which is important both for auditors and for long-term project maintenance. By following the practices of monitoring and controlling project work, it can be ensured that all assemblies are properly documented in accordance with the project's objectives and goals.

## [L-04] MISSING DOCSTRINGS
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the [Solidity official documentation](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html). In complex projects, the interpretation of all functions and their arguments and returns is important for code readability and auditability. 

### PROOF OF CONCEPT

// @audit The followings contracts are missing its @param @notice @return statement. Consider including it.

- [DNSClaimChecker](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnsregistrar/DNSClaimChecker.sol)
- [OffchainDNSResolver](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnsregistrar/OffchainDNSResolver.sol)
- [RecordParser](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnsregistrar/RecordParser.sol)
- [DNSRegistrar](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnsregistrar/DNSRegistrar.sol)
- [RSASHA1Algorithm](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/algorithms/RSASHA1Algorithm.sol)
- [EllipticCurve](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/algorithms/EllipticCurve.sol)
- [ModexpPrecompile](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/algorithms/ModexpPrecompile.sol)
- [RSASHA256Algorithm](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/algorithms/RSASHA256Algorithm.sol)
- [SHA1Digest](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/digests/SHA1Digest.sol)
- [SHA256Digest](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/digests/SHA256Digest.sol)
- [SHA1](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/SHA1.sol)

### MITIGATION
NatSpec comments should be increased in contracts. Follow your our best practice like [BytesUtils.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/BytesUtils.sol).

## [L-05] FUNCTION OVERLOADING
Having multiple functions with the same name in a smart contract can be dangerous or not a good practice for several reasons:

- Confusion: If there are several functions with the same name, it can be confusing for developers and users who are interacting with the smart contract. This can lead to errors and misunderstandings in the use of the contract.

- Security vulnerabilities: If multiple functions are defined with the same name, attackers can attempt to exploit this vulnerability to access or modify data or functionalities of the smart contract.

- Network overload: If there are multiple functions with the same name, there may be an impact on the efficiency and speed of the contract, as the network may be confused in trying to determine which function should be executed.

See more about this on [Solidity Docs.](https://docs.soliditylang.org/en/v0.8.19/contracts.html#function-overloading)

### PROOF OF CONCEPT
```solidity
contracts/dnssec-oracle/BytesUtils.sol
    32: function compare(
    52: function compare(
    111: function equals(
    129: function equals(
    148: function equals(
    164: function equals(

contracts/dnssec-oracle/DNSSECImpl.sol
    87: function verifyRRSet(
    107: function verifyRRSet(
```

## [L-06] WE SUGGEST USING THE OPENZEPPELIN SAFECAST LIBRARY
We have noticed that the contract `EllipticCurve.sol#L53-L56` and `BytesUtils.sol#L92` implements many type conversions from uint256 to int256. We recommend using the OpenZeppelin SafeCast library to make the project more robust and take advantage of the gas optimizations and best practices provided by OpenZeppelin.

```solidity
openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol#L290
    /**
     * @dev Returns the downcasted uint128 from uint256, reverting on
     * overflow (when the input is greater than largest uint128).
     *
     * Counterpart to Solidity's `uint128` operator.
     *
     * Requirements:
     *
     * - input must fit into 128 bits
     *
     * _Available since v2.5._
     */
    function toUint128(uint256 value) internal pure returns (uint128) {
        require(value <= type(uint128).max, "SafeCast: value doesn't fit in 128 bits");
        return uint128(value);
    }
```
## [N-01] MISSING VISIBILITY IN VARIABLES
In multiple instances throughout the scope, there are many variables that do not have an explicit visibility modifier. In the interest of clarity, consider including it.

### PROOF OF CONCEPT
```solidity
contracts/dnsregistrar/DNSClaimChecker.sol
16: uint16 constant CLASS_INET = 1;
17: uint16 constant TYPE_TXT = 16;

contracts/dnssec-oracle/RRUtils.sol
72: uint256 constant RRSIG_TYPE = 0;
73: uint256 constant RRSIG_ALGORITHM = 2;
74: uint256 constant RRSIG_LABELS = 3;
75: uint256 constant RRSIG_TTL = 4;
76: uint256 constant RRSIG_EXPIRATION = 8;
77: uint256 constant RRSIG_INCEPTION = 12;
78: uint256 constant RRSIG_KEY_TAG = 16;
79: uint256 constant RRSIG_SIGNER_NAME = 18;
210: uint256 constant DNSKEY_FLAGS = 0;
211: uint256 constant DNSKEY_PROTOCOL = 2;
212: uint256 constant DNSKEY_ALGORITHM = 3;
213: uint256 constant DNSKEY_PUBKEY = 4;
236: uint256 constant DS_KEY_TAG = 0;
237: uint256 constant DS_ALGORITHM = 2;
238: uint256 constant DS_DIGEST_TYPE = 3;
239: uint256 constant DS_DIGEST = 4;
```
And so much more instances in the following contracts:
- [EllipticCurve.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/algorithms/EllipticCurve.sol#L21-L35)
- [DNSSECImpl.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/DNSSECImpl.sol#L27-L32)


## [N-02] USING WHILE FOR UNBOUNDED LOOPS ISN'T RECOMMENDED
Improve the efficiency and stability of your code by avoiding unbounded `while loops`. Instead of using a `while loop` for unbounded iterations, it is recommended to use other loop structures like `for` that have a clearer structure and can provide better control flow for the loop. Don't write loops that are unbounded as this can hit the gas limit, causing your transaction to fail. For the reason above, while and do-while loops are rarely used.

### PROOF OF CONCEPT
```solidity
// @audit Avoid unbounded loops
    contracts/dnsregistrar/DNSClaimChecker.sol
        51: while (idx < endIdx) {

    contracts/dnssec-oracle/RRUtils.sol
        24: while (true) {
        60: while (true) {
        267: while (counts > othercounts) {
        291: while (counts > othercounts) {
        297: while (othercounts > counts) {
        304: while (counts > 0 && !self.equals(off, other, otheroff)) {

    contracts/dnssec-oracle/algorithms/EllipticCurve.sol
        51:  while (r2 != 0) {
```
### RECOMMENDATION
- Avoid unbounded loops: As mentioned before, it is important to avoid unbounded while loops in your smart contracts. If you need to make a loop, make sure that the number of iterations is limited and known in advance.


## [N-03] SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE
Short-circuiting is a solidity contract development model that uses `OR/AND` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

### PROOF OF CONCEPT
```solidity
contracts/dnsregistrar/OffchainDNSResolver.sol
86:  if (!rrname.equals(name) || iter.class != CLASS_INET || iter.dnstype != TYPE_TXT)
144: if (txt.length < 5 || !txt.equals(0, "ENS1 ", 0, 5))
178: if (nameOrAddress[idx] == "0" && nameOrAddress[idx + 1] == "x")

contracts/dnsregistrar/DNSRegistrar.sol
129: interfaceID == type(IERC165).interfaceId || interfaceID == type(IDNSRegistrar).interfaceId;
187: if (owner == address(0) || owner == previousRegistrar)

contracts/dnssec-oracle/BytesUtils.sol
154: self.length == offset + other.length && equals(self, offset, other, 0, other.length);
169: self.length == other.length && equals(self, 0, other, 0, self.length);

contracts/dnssec-oracle/RRUtils.sol
304: while (counts > 0 && !self.equals(off, other, otheroff))

contracts/dnssec-oracle/algorithms/RSASHA1Algorithm.sol
44: return ok && SHA1.sha1(data) == result.readBytes20(result.length - 20);

contracts/dnssec-oracle/algorithms/EllipticCurve.sol
42:  if (u == 0 || u == m || m == 0) return 0;
128: if (x0 == 0 && y0 == 0)
138: if (0 == x || x == p || 0 == y || y == p)
392: if (rs[0] == 0 || rs[0] >= n || rs[1] == 0)

contracts/dnssec-oracle/DNSSECImpl.sol
201: if (name.length != iter.data.nameLength(iter.offset) || !name.equals(0, iter.data, iter.offset, name.length))
```

## [N-04] TAKE ADVANTAGE OF CUSTOM ERROR'S RETURN VALUE PROPERTY
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the `()` sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

### PROOF OF CONCEPT
```solidity
contracts/dnsregistrar/DNSRegistrar.sol
152:  if (!RRUtils.serialNumberGte(inception, inceptions[node])) { revert StaleProof();}
159:  if (!found) { revert NoOwnerRecordFound();}

contracts/dnssec-oracle/DNSSECImpl.sol
201: if (name.length != iter.data.nameLength(iter.offset) || !name.equals(0, iter.data, iter.offset, name.length)) { revert InvalidRRSet();}
```

## [N-05] CONTRACT DOES NOT FOLLOW THE SOLIDITY STYLE GUIDE'S SUGGESTED LAYOUT ORDERING
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be: 
- 1. Type declarations 
- 2. State variables,
- 3. Events 
- 4. Modifiers 
- 5. Functions 
but the contract(s) below do not follow this ordering.

- [RRUtils.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/RRUtils.sol)
- [DNSRegistrar.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnsregistrar/DNSRegistrar.sol)


## [N-06] MISSING SPDX LICENSE IDENTIFIER
Trust in smart contracts can be better established if their source code is available. Since making source code available always touches on legal problems with regards to copyright, the Solidity compiler encourages the use of machine-readable SPDX license identifiers. Every source file should start with a comment indicating its license.
[See the solidity docs](https://docs.soliditylang.org/en/v0.8.19/layout-of-source-files.html#spdx-license-identifier)

The contract(s) below do not follow this requirement:
- [BytesUtils.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/BytesUtils.sol)



## [R-01] FUNCTION NAMING SUGGESTIONS (Underscore Prefix for Non-external Functions and Variables)
Proper use of _ as a function name prefix and a common pattern is to prefix internal and private function names with _. This pattern is correctly applied in all contracts, however there are some inconsistencies in just these contracts.

### PROOF OF CONCEPT
```solidity
// @audit Refactorize
    contracts/dnsregistrar/DNSClaimChecker.sol
        19:  function getOwnerAddress(
                bytes memory name,
                bytes memory data
            ) internal pure returns (address, bool) {

        46: function parseRR(
                bytes memory rdata,
                uint256 idx,
                uint256 endIdx
            ) internal pure returns (address, bool) {

        66: function parseString(
                bytes memory str,
                uint256 idx,
                uint256 len
            ) internal pure returns (address, bool) {

    contracts/dnsregistrar/OffchainDNSResolver.sol
        136: function parseRR(
                bytes memory data,
                uint256 idx,
                uint256 lastIdx
            ) internal view returns (address, bytes memory) {

        162: function readTXT(
                bytes memory data,
                uint256 startIdx,
                uint256 lastIdx
            ) internal pure returns (bytes memory) {

        173: function parseAndResolve(
                bytes memory nameOrAddress,
                uint256 idx,
                uint256 lastIdx
            ) internal view returns (address) {

        190: function resolveName(
                bytes memory name,
                uint256 idx,
                uint256 lastIdx
            ) internal view returns (address) {

        209: function textNamehash(
                bytes memory name,
                uint256 idx,
                uint256 lastIdx
            ) internal view returns (bytes32) {

    contracts/dnsregistrar/RecordParser.sol
        14: function readKeyValue(
                bytes memory input,
                uint256 offset,
                uint256 len
            )
                internal
                pure
                returns (bytes memory key, bytes memory value, uint256 nextOffset)
            {

    contracts/dnssec-oracle/BytesUtils.sol
        13: function keccak(
                bytes memory self,
                uint256 offset,
                uint256 len
            ) internal pure returns (bytes32 ret) {

        32: function compare(
                bytes memory self,
                bytes memory other
            ) internal pure returns (int256) {

        52: function compare(
                bytes memory self,
                uint256 offset,
                uint256 len,
                bytes memory other,
                uint256 otheroffset,
                uint256 otherlen
            ) internal pure returns (int256) {

        111: function equals(
                bytes memory self,
                uint256 offset,
                bytes memory other,
                uint256 otherOffset,
                uint256 len
            ) internal pure returns (bool) {

        129:  function equals(
                bytes memory self,
                uint256 offset,
                bytes memory other,
                uint256 otherOffset
            ) internal pure returns (bool) {

        148:  function equals(
                bytes memory self,
                uint256 offset,
                bytes memory other
            ) internal pure returns (bool) {

        164: function equals(
                bytes memory self,
                bytes memory other
            ) internal pure returns (bool) {

        179: function readUint8(
                bytes memory self,
                uint256 idx
            ) internal pure returns (uint8 ret) {

        192: function readUint16(
                bytes memory self,
                uint256 idx
            ) internal pure returns (uint16 ret) {

        208: function readUint32(
                bytes memory self,
                uint256 idx
            ) internal pure returns (uint32 ret) {

        224: function readBytes32(
                bytes memory self,
                uint256 idx
            ) internal pure returns (bytes32 ret) {

        240: function readBytes20(
                bytes memory self,
                uint256 idx
            ) internal pure returns (bytes20 ret) {

        260: function readBytesN(
                bytes memory self,
                uint256 idx,
                uint256 len
            ) internal pure returns (bytes32 ret) {

        300: function substring(
                bytes memory self,
                uint256 offset,
                uint256 len
            ) internal pure returns (bytes memory) {

        332: function base32HexDecodeWord(
                bytes memory self,
                uint256 off,
                uint256 len
            ) internal pure returns (bytes32) {

        387: function find(
                bytes memory self,
                uint256 off,
                uint256 len,
                bytes1 needle
            ) internal pure returns (uint256) {

    contracts/dnssec-oracle/RRUtils.sol
        19:     function nameLength(
                bytes memory self,
                uint256 offset
            ) internal pure returns (uint256) {
            
        41: function readName(
                bytes memory self,
                uint256 offset
            ) internal pure returns (bytes memory ret) {

        55: function labelCount(
                bytes memory self,
                uint256 offset
            ) internal pure returns (uint256) {

        94: function readSignedSet(
                bytes memory data
            ) internal pure returns (SignedSet memory self) {

        111: function rrs(
                SignedSet memory rrset
            ) internal pure returns (RRIterator memory) {

        136: function iterateRRs(
                bytes memory self,
                uint256 offset
            ) internal pure returns (RRIterator memory ret) {

        150: function done(RRIterator memory iter) internal pure returns (bool) {
        158: function next(RRIterator memory iter) internal pure {
        187: function name(RRIterator memory iter) internal pure returns (bytes memory) {
        200: function rdata(
                RRIterator memory iter
            ) internal pure returns (bytes memory) {

        222: function readDNSKEY(
                bytes memory data,
                uint256 offset,
                uint256 length
            ) internal pure returns (DNSKEY memory self) {

        248: function readDS(
                bytes memory data,
                uint256 offset,
                uint256 length
            ) internal pure returns (DS memory self) {

        259: function isSubdomainOf(
                bytes memory self,
                bytes memory other
            ) internal pure returns (bool) {

        275: function compareNames(
                bytes memory self,
                bytes memory other
            ) internal pure returns (int256) {

        332: function serialNumberGte(
                uint32 i1,
                uint32 i2
            ) internal pure returns (bool) {

        341: function progress(
                bytes memory body,
                uint256 off
            ) internal pure returns (uint256) {

        353: function computeKeytag(bytes memory data) internal pure returns (uint16) {

    contracts/dnssec-oracle/algorithms/EllipticCurve.sol
        40: function inverseMod(uint256 u, uint256 m) internal pure returns (uint256) {
        65: function toProjectivePoint(
                uint256 x0,
                uint256 y0
            ) internal pure returns (uint256[3] memory P) {

        77: function toProjectivePoint(
                uint256 x0,
                uint256 y0
            ) internal pure returns (uint256[3] memory P) {

        92: function toAffinePoint(
                uint256 x0,
                uint256 y0,
                uint256 z0
            ) internal pure returns (uint256 x1, uint256 y1) {

        106:  function zeroProj()
                internal
                pure
                returns (uint256 x, uint256 y, uint256 z)
            {

        117: function zeroAffine() internal pure returns (uint256 x, uint256 y) {
        124: function isZeroCurve(
                uint256 x0,
                uint256 y0
            ) internal pure returns (bool isZero) {

        137: function isOnCurve(uint256 x, uint256 y) internal pure returns (bool) {
        159: function twiceProj(
                uint256 x0,
                uint256 y0,
                uint256 z0
            ) internal pure returns (uint256 x1, uint256 y1, uint256 z1) {

        208: function addProj(
                uint256 x0,
                uint256 y0,
                uint256 z0,
                uint256 x1,
                uint256 y1,
                uint256 z1
            ) internal pure returns (uint256 x2, uint256 y2, uint256 z2) {

        286: function add(
                uint256 x0,
                uint256 y0,
                uint256 x1,
                uint256 y1
            ) internal pure returns (uint256, uint256) {

        302: function twice(
                uint256 x0,
                uint256 y0
            ) internal pure returns (uint256, uint256) {
            
        316: function multiplyPowerBase2(
                uint256 x0,
                uint256 y0,
                uint256 exp
            ) internal pure returns (uint256, uint256) {

        335: function multiplyScalar(
                uint256 x0,
                uint256 y0,
                uint256 scalar
            ) internal pure returns (uint256 x1, uint256 y1) {
                if (scalar == 0) {

        337: function multipleGeneratorByScalar(
                uint256 scalar
            ) internal pure returns (uint256, uint256) {

        386: function validateSignature(
                bytes32 message,
                uint256[2] memory rs,
                uint256[2] memory Q
            ) internal pure returns (bool) {

    contracts/dnssec-oracle/algorithms/P256SHA256Algorithm.sol
        30: function parseSignature(
                bytes memory data
            ) internal pure returns (uint256[2] memory) {

        37: function parseKey(
                bytes memory data
            ) internal pure returns (uint256[2] memory) {

    contracts/dnssec-oracle/algorithms/ModexpPrecompile.sol
        7: function modexp(
                bytes memory base,
                bytes memory exponent,
                bytes memory modulus
            ) internal view returns (bool success, bytes memory output) {

    contracts/dnssec-oracle/algorithms/RSAVerify.sol
        14: function rsarecover(
                bytes memory N,
                bytes memory E,
                bytes memory S
            ) internal view returns (bool, bytes memory) {

    contracts/dnssec-oracle/DNSSECImpl.sol
        140: function validateSignedSet(
                RRSetWithSignature memory input,
                bytes memory proof,
                uint256 now
            ) internal view returns (RRUtils.SignedSet memory rrset) {

        181: function validateRRs(
                RRUtils.SignedSet memory rrset,
                uint16 typecovered
            ) internal pure returns (bytes memory name) {

        225: function verifySignature(
                bytes memory name,
                RRUtils.SignedSet memory rrset,
                RRSetWithSignature memory data,
                bytes memory proof
            ) internal view {

        254: function verifyWithKnownKey(
                RRUtils.SignedSet memory rrset,
                RRSetWithSignature memory data,
                RRUtils.RRIterator memory proof
            ) internal view {

        285: function verifySignatureWithKey(
                RRUtils.DNSKEY memory dnskey,
                bytes memory keyrdata,
                RRUtils.SignedSet memory rrset,
                RRSetWithSignature memory data
            ) internal view returns (bool) {

        330: function verifyWithDS(
                RRUtils.SignedSet memory rrset,
                RRSetWithSignature memory data,
                RRUtils.RRIterator memory proof
            ) internal view {

        373: function verifyKeyWithDS(
                bytes memory keyname,
                RRUtils.RRIterator memory dsrrs,
                RRUtils.DNSKEY memory dnskey,
                bytes memory keyrdata
            ) internal view returns (bool) {

        415: function verifyDSHash(
                uint8 digesttype,
                bytes memory data,
                bytes memory digest
            ) internal view returns (bool) {

    contracts/dnssec-oracle/SHA1.sol
        6: function sha1(bytes memory data) internal pure returns (bytes20 ret) {

    contracts/utils/NameEncoder.sol
        9: function dnsEncodeName(
                string memory name
            ) internal pure returns (bytes memory dnsName, bytes32 node) {

    contracts/utils/HexUtils.sol
        11: function hexStringToBytes32(
                bytes memory str,
                uint256 idx,
                uint256 lastIdx
            ) internal pure returns (bytes32 r, bool valid) {

        68: function hexToAddress(
                bytes memory str,
                uint256 idx,
                uint256 lastIdx
            ) internal pure returns (address, bool) {
```
### RECOMMENDATION
- `_singleLeadingUnderscore`

This convention is suggested for non-external functions and state variables (private or internal). State variables without a specified visibility are internal by default.

When designing a smart contract, the public-facing API (functions that can be called by any account) is an important consideration. Leading underscores allow you to immediately recognize the intent of such functions, but more importantly, if you change a function from non-external to external (including public) and rename it accordingly, this forces you to review every call site while renaming. This can be an important manual check against unintended external functions and a common source of security vulnerabilities (avoid find-replace-all tooling for this change).

```diff
-19: function getOwnerAddress(
                bytes memory name,
                bytes memory data
            ) internal pure returns (address, bool) {
+19: function _getOwnerAddress(
                bytes memory name,
                bytes memory data
            ) internal pure returns (address, bool) {
```

## [N-07] MANDATORY CHECKS FOR EXTRA SAFETY IN THE SETTERS
In the folowing function below, there is a check that can be made in order to achieve more safe and efficient code.

Address zero check can be added in the function setPublicSuffixList.

### PROOF OF CONCEPT
```solidity
@audit Check that it's not address 0
contracts/dnsregistrar/DNSRegistrar.sol
80:  function setPublicSuffixList(PublicSuffixList _suffixes) public onlyOwner {
        suffixes = _suffixes;
        emit NewPublicSuffixList(address(suffixes));
    }              
```

## [N-08] ALSO VALUES IN ASSEMBLY CAN BE STORED IN CONSTANT VARIABLES
Values in assembly can also benefit from constant variables, which would also increase the readability of these functions

### PROOF OF CONCEPT
```solidity
@audit 32
    contracts/dnssec-oracle/BytesUtils.sol
        73: assembly {
                    selfptr := add(self, add(offset, 32))
                    otherptr := add(other, add(otheroffset, 32))
                }

        257:  assembly {
                    let mask := not(sub(exp(256, sub(32, len)), 1))
                    ret := and(mload(add(add(self, 32), idx)), mask)
                }

        311: assembly {
                    dest := add(ret, 32)
                    src := add(add(self, 32), offset)
                }
            
    contracts/dnssec-oracle/algorithms/ModexpPrecompile.sol
        23:  assembly {
                    success := staticcall(
                        gas(),
                        5,
                        add(input, 32),
                        mload(input),
                        add(output, 32),
                        mload(modulus)
                    )
                }

```
@audit In this contract as well 
https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/SHA1.sol

## [N-09] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
In the contracts `DNSClaimChecker.sol`, `OffchainDNSResolver.sol`, `RRUtils.sol`, `EllipticCurve.sol`, `DNSSECImpl.sol` there are many `constanst` used in the system. It is recommended to put the most used ones in one file (for example `Constants.sol`) and use inheritance to access these values.

This helps with readability and easier maintenance for future changes. It also helps with any issues, as some of these hard-coded contracts are admin contracts.

`Constants.sol` Use and import these files in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

### PROOF OF CONCEPT
```solidity
contracts/dnsregistrar/DNSClaimChecker.sol
// @audit Create a specific file for these cases and use inheritance.
    16: uint16 constant CLASS_INET = 1;
    17: uint16 constant TYPE_TXT = 16;

contracts/dnsregistrar/OffchainDNSResolver.sol
    29: uint16 constant CLASS_INET = 1;
    30: uint16 constant TYPE_TXT = 16;

contracts/dnssec-oracle/RRUtils.sol
    72: uint256 constant RRSIG_TYPE = 0;
    73: uint256 constant RRSIG_ALGORITHM = 2;
    74: uint256 constant RRSIG_LABELS = 3;
    75: uint256 constant RRSIG_TTL = 4;
    76: uint256 constant RRSIG_EXPIRATION = 8;
    77: uint256 constant RRSIG_INCEPTION = 12;
    78: uint256 constant RRSIG_KEY_TAG = 16;
    79: uint256 constant RRSIG_SIGNER_NAME = 18;
    210: uint256 constant DNSKEY_FLAGS = 0;
    211: uint256 constant DNSKEY_PROTOCOL = 2;
    212: uint256 constant DNSKEY_ALGORITHM = 3;
    213: uint256 constant DNSKEY_PUBKEY = 4;
    236: uint256 constant DS_KEY_TAG = 0;
    237: uint256 constant DS_ALGORITHM = 2;
    238: uint256 constant DS_DIGEST_TYPE = 3;
    239: uint256 constant DS_DIGEST = 4;

contracts/dnssec-oracle/algorithms/EllipticCurve.sol
    21: uint256 constant a =
            0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFC;
    23: uint256 constant b =
            0x5AC635D8AA3A93E7B3EBBD55769886BC651D06B0CC53B0F63BCE3C3E27D2604B;
    25: uint256 constant gx =
            0x6B17D1F2E12C4247F8BCE6E563A440F277037D812DEB33A0F4A13945D898C296;
    27: uint256 constant gy =
            0x4FE342E2FE1A7F9B8EE7EB4A7C0F9E162BCE33576B315ECECBB6406837BF51F5;
    29:uint256 constant p =
            0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFF;
    31: uint256 constant n =
            0xFFFFFFFF00000000FFFFFFFFFFFFFFFFBCE6FAADA7179E84F3B9CAC2FC632551;

    34: uint256 constant lowSmax =
            0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0;

contracts/dnssec-oracle/DNSSECImpl.sol
    27: uint16 constant DNSCLASS_IN = 1;
    29: uint16 constant DNSTYPE_DS = 43;
    30: uint16 constant DNSTYPE_DNSKEY = 48;
    32: uint256 constant DNSKEY_FLAG_ZONEKEY = 0x100;
```
### RECOMMENDATION
We strongly recommend implementing the constanst variables in a separate file. This will help you avoid errors and confusion in your project, and maintain a more organized and readable code structure. Following these good programming and organizational practices will help you build a solid and scalable project, which will be easier to maintain and update in the future.

## [R-02] SHORTHAND WAY TO WRITE IF/ELSE STATEMENT
The regular if-else statement can be refactored in an abbreviated way to write it. This increases readability and reduces the overall SLOC (lines of code).

```solidity
contracts/dnsregistrar/OffchainDNSResolver.sol
150:    if (lastTxtIdx > txt.length) {
            address dnsResolver = parseAndResolve(txt, 5, txt.length);
            return (dnsResolver, "");
        } else {
            address dnsResolver = parseAndResolve(txt, 5, lastTxtIdx);
216:    if (separator < lastIdx) {
            parentNode = textNamehash(name, separator + 1, lastIdx);
        } else {
            separator = lastIdx;
        }            

contracts/dnssec-oracle/BytesUtils.sol
87:     if (shortest - idx >= 32) {
                    mask = type(uint256).max;
        } else {
                    mask = ~(2 ** (8 * (idx + 32 - shortest)) - 1);
        }

```
And so much more instances in the following contracts:
- [DNSRegistrar.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnsregistrar/DNSRegistrar.sol#L188-L200)
- [RSASHA1Algorithm.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/algorithms/RSASHA1Algorithm.sol#L23-L36)
- [EllipticCurve.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/algorithms/EllipticCurve.sol#L233-L239)
- [RSASHA256Algorithm.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/dnssec-oracle/algorithms/RSASHA256Algorithm.sol#L22-L35)
- [NameEncoder.sol](https://github.com/code-423n4/2023-04-ens/blob/main/contracts/utils/NameEncoder.sol#L26-L38)

## [S-01] WE SUGGEST TO USE NAMED PARAMETERS FOR MAPPING TYPE DECLARATIONS
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.
```solidity
mapping(address account => uint256 balance); 
```
