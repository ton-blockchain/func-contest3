# Welcome to the TON Smart Challenge #3 by the TON Foundation

There are five tasks listed below
1. Large cell builder
2. Two parameter queue
3. Text calculator
4. Curve25519 operations
5. Mini Elector

Each tasks have two parts:

* A comment with a description of what the smart contract should do.

* The code of the smart contract with one or more functions marked as `testable`.

The goal of the contestants is to provide code that match the description.
Each task may give contestant either 0 or 5 to 6 score points: 5 for all tests passed plus "gas-score" from 0 to 1 (0 for "infinite" gas consumption, 1 for 0 gas consumption, dependence is inverse exponent).
Each tvm execution is limited by 100,000,000 (hundred millions) gas units.
This limit is high enough that it only rules out infinite loops. Any practical solution, regardless of how (un)optimized it is, will fit.

We ask participants not to change the signature (number, order, and types of arguments and result) of `testable` functions for us to be able to evaluate their submission.

# Solutions open-sourced by participants
[crazyministr](https://github.com/crazyministr/TonContest-FunC/tree/master/func-contest3)

[nns2009](https://github.com/nns2009/TON-FunC-contest-3)

[shuva10v](https://github.com/shuva10v/func-contest3-solutions)

# Stages
There are two mutually independent stages of contest, check more info on [challenge website](https://ton.org/en/ton-smart-challenge-3).



## How to submit

Tasks are expected to be submitted through [@toncontests_bot](https://t.me/toncontests_bot).

Solutions should be submitted as a zip archive that contains up to 6 files labeled `1.fc`, `2.fc`, `3.fc`, `4.fc`, `5.fc` (each file representing a corresponding task), and `participant.json`.

Each file will be compiled with FunC from the [main repository](https://github.com/ton-blockchain/ton/tree/master/crypto/func).
All solutions from the contestants will be linked with stdlib.fc (from the [ton-blochcain repository](https://github.com/ton-blockchain/ton/blob/master/crypto/smartcont/stdlib.fc)).

`participant.json` should have the following structure:
```
{
  "address" : "your-TON-wallet-address", 
  "username": "desired-username-in-public-list",
  "codeforces": "(optional)codeforces-username"
}
```

The participant can send solutions and receive the result after a short evaluation delay any number of times, but not more than five times per hour. The best solution submitted (with the highest total score of all the five tasks sent in one archive) will be used to determine the final rank. The organizers of the competition reserve the right to publish participants’ solutions with usernames (decided by participants themselves) after the contest.

# How will it be checked
For each problem, we have a set of test vectors which satisfy the description.

We will automatically run those tests against the testable functions.

If functions of one task pass all the tests, this task is considered "solved".

For "solved" tasks gas-usage of each test is recorded, the sum of gas used by all tests is the second-order metric which will allow us to determine the winner amongst people with equal number of solved tasks.

Winners are determined as follows:

* Person who solved more tasks will be ranked higher than person who will solve less (regardless of optimization).

* For evaluation of gas-usage each problem has a weight which in general inversely correlated with average gas-usage, thus 10 gas units optimization of the function which consume 100 gas units will be scored higher than 10 gas units optimization of the function which consume 10000 gas units.

# Tasks
## 1. Large cell builder
```func
{-
  In TON there is a limit on the size of the external message which can be sent equal to 64 kB. Sometimes it is necessary to send a larger message; it requires the onchain construction of one message from multiple smaller parts. Your task is to create such construction contract.
  In particular, a contestant needs to develop a FunC contract with two features:
    a) it has get_method "decomposite" for decomposition of large cell to parts: it accepts 1 cell (number_of_bits<1000000, number_of_cells<4000 , depth<256) and 1 address and returns tuple of cells (each of which has less than 1000 distinct cells and 40000 bits total), those cells will be transformed to slice and sent as internal message body to the contract.
    b) recv_internal should handle those internal messages from get-method described above and upon receiving last one, send initial large cell to the address (coins amount 0, mode 0). For simplicity, it is guaranteed that messages will be sent exactly in the order in which they were in decomposite output and no other messages will be sent in between.
  Note, that initial state of contract storage will be empty cell: cell with zero bits and refs.
  
  It is necessary to stress the deduplication mechanism of dag (or bag) of cells. If exactly the same cell (that means both same bits and same refs) is appeared in different part of the dag twice or multiple times it will not be stored or counted separately. All numbers in the task like 64kb, number_of_bits<1000000, number_of_cells<4000, 1000 cells and 40000 bits in output is given with account for deduplication. FunC functions compute_data_size/slice_compute_data_size (as well as underlying opcodes CDATASIZE/SDATASIZE) returns output with account for deduplication as well.
  
-}

;; testable
() recv_internal (slice body) {
}

;; testable
tuple decomposite (cell big_cell, slice destination_address) method_id {
}
```



## 2. Two parameter queue
```func
{-
  Contract handles internal messages with queries with the following scheme
  `_# score:uint32 value:(VarUInteger 16) msg:^Cell = MsgInternalBody`, where msg contains message body which shoud be sent later and store it to contract.
  Once the number of stored queries reaches 12, contract should send and delete from storage message with the highest score and message with the lowest value (if it is the same message, it should be sent once). Messages should be sent to any address with mode 0, coin amount should be equal to value and it should contain corresponding message body. All scores and values are guaranteed to be different
  Note, that in addition to gas-fees, storage fees will be used to determine final score. In particular, storage fee will be calculated like between each message passes 3 days (259200 seconds). Gas-units price and storage fee params will correspond to current configs of masterchain: 1000 nanoTON per 65536 bits per second + 500000 nanoTON per 65536 cells per second; gas is 10000 nanoTON per unit.


  Example:
  (message with score x and value y are represented as `(x,y)` )

  incoming message   outcoming messages     
  (1, 5)           | -
  (2, 6)           | -
  (3, 100)         | -
  (4, 2)           | -
  (5, 3)           | -
  (6, 4)           | -
  (7, 7)           | -
  (8, 8)           | -
  (9, 9)           | -
  (10, 10)         | -
  (11, 11)         | -
  (12, 20)         | (12,20); (4,2)
  (15, 1)          | -
  (13, 13)         | (15, 1)
  (14, 14)         | (14,14); (5,3)
-}

;; testable
() recv_internal (slice in_msg_body) {
  (int score, int value, cell msg) = (in_msg_body~load_uint(32), in_msg_body~load_coins(), in_msg_body~load_ref());
  ;; Add code here
}
```

## 3. Text calculator
```func
  {-
  Contract receives internal message with text comment (https://ton.org/docs/develop/smart-contracts/guidelines/internal-messages) which contains arithmetic expression containing integer numbers in decimal representation and operations `(+-*/)`.
  All values (including intermediate) fit 256 bit. Contract should respond (coins = 0, mode = 64) with correct answer encoded as text comment back.
  It is guaranteed that all tests contain a valid algebraic equations.
  Division result should be rounded down. It is guaranteed that tests do not contain division by zero.
  -}

;; testable
() recv_internal (cell message, slice in_msg_body) {
  ;; Add code here
}
```

## 4. Curve25519 operations
```func
{-
  Implement Curve25519 addition and multiplication.
-}

() recv_internal () {
}

;; testable
(int,int) add(int x1, int y1, int x2, int y2) {
  ;; Add code here
  ;; return x,y coordinate of Point1 + Point2
}

;; testable
int mul(int x1, int factor) {
  ;; Add code here
  ;; return x coordinate of Point1 * factor
}
```
For `x1=56391866308239752110494101482511933051315484376135027248208522567059122930692`, `y1=17671033459111968710988296061676524036652749365424210951665329683594356030064` and `x2=39028180402644761518992797890514644768585183933988208227318855598921766377692`, `y2=17694324391104469229766971147677885172552105420452910290862122102896539285628`  result of `add(x1, y1, x2, y2)` expected to be `7769460008531208039267550090770832052561793182665100660016059978850497673345, 50777594312607721283178588283812137388073334114015585272572035433724485979392`. Tests only checks additions where the result of the addition is the finite valid point on the curve.

For `x1=56391866308239752110494101482511933051315484376135027248208522567059122930692` and `factor=4`, result of `mul(x1,factor)` expected to be `41707806908216107150933211614905026312154955484464515789593741233629885877574`.

## 5. Mini Elector
```func
{-
  Validators in TON network are chosen onchain by special smart-contract called Elector: participants sends their application and smart-contract deterministically decides who will be the next validator. Your task is to implement (in simplified form) election logic in the gas-optimal way:
  "Mini-elector" should accept internal messages with the following layout.
    a) `participate#5ce28eea query_id:uint64 max_factor:uint24 = InternalMsgBody;`. Upon receiving this message contract should store sender of the message (called key), max_factor and amount of TON attached to message (called stake) to storage (if key already exists in the table max_factor should be rewritten to new one while amount should be added to previously processed). If maxfactor is less than 65536 it should be treated as equal to 65536, if maxfactor is higher than 655360 it should be treated as equal to 655360.
    b) `try_elect#207fa5f5 query_id:uint64 = InternalMsgBody;` - upon receiving this message contract should try to form winners list (key, effective_stake) from participant' applications. Note that `effective_stake` may be less than `stake` (in other words, not all the stake will work). Excesses of the stake (as well as stakes of "losers", should be stored separately).
    Rules of forming a list:
      I) it has at least 5 rows
      II) for each participant A in the list, the ratio of A's `effective_stake` to the `effective_stake` of participant with the smallest stake `effective_stake` should be equal or less to A's max_factor/65536 (in other words, max_factor is 65536 based rational number).
      III) Under conditions I and II, total effective stake (sum of `effective_stake` of all winners) should be maximal.

    If it is not possible to form a list, contract should throw. Otherwise, it should respond with 
    `success#eefa5ea4 query_id:uint64 total_winners:uint32 total_effective_stake:(VarUInteger 16) unused_stake:(VarUInteger 16) = InternalMsgBody;` (query_id matched that in try_elect)

    After responding with `success` message, contract's get_method `get_stake_table` should return two tuples with winners and "unused funds", this tuples should contain exactly the same number of elements as there are winners/participants-with-unused-funds (NOT lisp-style lists), each element should be in format [address-as-a-slice, stake-as-number].  Note that if participants' stake is not fully used, it will be presented in both tuples. For instance, possible result of get_stake_table can be `(["Ef8RERERERERERERERERERERERERERERERERERERERERERlb"a, 10], ["Ef8iIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiIiImKK"a, 1],  ["Ef8zMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzMzM0vF"a, 1], ["Ef9ERERERERERERERERERERERERERERERERERERERERERJUo"a, 1],  ["Ef9VVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVbxn"a, 1]), (["Ef8RERERERERERERERERERERERERERERERERERERERERERlb"a, 10])`.
    
    Note that tests are organized as following: there will be a few participate requests (less than 255) followed by one try_elect and then response and get_method result will be checked.
-}

;; testable
() recv_internal (int msg_value, cell full_message, slice in_msg_body) {
  (int op, int query_id) = (in_msg_body~load_uint(32),in_msg_body~load_uint(64));
  if(op == 0x5ce28eea) {
   ....
  }
  if(op == 0x207fa5f5) {
   ....
  }
}

;; testable
(tuple, tuple) get_stake_table() method_id {
}
```

## Docs

We recommend participants start from here:
* [Developers Portal](https://ton.org/en/dev)
* [TON Documentation](https://ton.org/docs)

The most relevant places to start in the documentation:
* [TON concepts](https://ton.org/docs/learn/overviews/TON_blockchain_overview)
* [Develop Smart Contracts](https://ton.org/docs/develop/smart-contracts/)

Additional and detailed information is available in the [whitepapers](https://ton.org/docs/learn/docs).

Examples of standard smart contracts can be found [here](https://github.com/ton-blockchain/ton/tree/master/crypto/smartcont).

Developer Chats - [@tondev_eng](https://t.me/tondev_eng), [@tondev](https://t.me/tondev).

FunC studying chat - [@ton_learn](https://t.me/ton_learn).

Introduction and tutorials for FunC are available here:
* https://ton.org/docs/develop/func/overview

## How to compile and test

We recommend using the [toncli](https://ton.org/docs/develop/smart-contracts/testing/toncli) or [ton-contract-executor](https://github.com/Naltox/ton-contract-executor) - open-source tools written by community developers.

These tools allow you to work with FunC smart contracts, compile them, and run local tests.

If for some reason you don’t want to use the tool, you can use the FunC compiler and Fift scripts [directly](https://ton.org/docs/develop/howto/compile#func).

For syntax highlighting, you can use [IDE plugins](https://ton.org/docs/develop/smart-contracts/environment/ide-plugins).
