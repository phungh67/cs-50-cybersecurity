
# 1. Definition

>[! Shaw's Methods]
>The estimated WCET is the execution time of the longest structural path through the program.

# 2. A simple example

```C++
for (i=1; i <= N; i++){
	if (A > K)
		A = K - 1; // T1
	else
		A = K + 1; // E1
		
	if (A < K)
		A = K; // T2
	else
		A = K - 1; // E2
}
```

In the above example, we have a loop structure with some condition check. There are two main branches: one to check if A is larger than K and the other branch is to handler is A less than K.

>[! Notes]
>The estimated WCET (WCETe) is the execution time of the ***longest structural*** path through the program.

For example, the formula for this example is:
$$
WCETe = N * (WCET(loop) + WCET(I_1) + max(WCET(T_1), WCET(E_1)) + WCET(I_2) + max(WCET(T_2), WCET(E_2)))
$$

There are several things to consider:
- The total size of the loop (N).
- The cost of the loop command itself (not the total, but the cost to invoke the loop).
- The cost of first condition (A > K).
- Worst path of first condition (True or Error), worst path of second condition (True of Error).

# 3. Exercises
## 3.1 Theory practice

>[!Problem 1]
>WCET analysis: we will find out how long does it take for the program to travel with the longest path in its execution flow.

**Problem**: consider the famous Fibonacci function `Caculate()`
```C
int Calculate (int Z) {
	int R;
	if (Z == 0)
		R = 1;
	else if (Z == 1)
		R = 1;
	else
		R = Calculate(Z-1) + Calculate(Z-2);\
	return R;
}
```
1. The requirement is using the Shaw's method, we need to estimate the WCET for the function with a given value of Z (integer) -> lit. a formula with variable 'Z'
	- Each *declaration* or *assignment* statement costs 1 time unit.
	- Each *compare* costs 1 time unit.
	- *return* also costs 1 time unit.
	- *addition* and *subtraction* costs 4 time units.
	- A function call takes 2 time units plus WCET for the function in question ( 2 + ...).
	- All other language constructs can be assumed as 0 time units to execute.

---
>[! First question]
>Using Shaw's method to estimate the WCET for function `Calculate()`in term of Z?

From top to bottom, for each condition check, it also costs too. Then we have this procedure:
- Initialize of `R`= 1
- Perform condition check (twice) because we consider the worst case.
- Assign value for `R`costs 1, the addition of two recursive functions is 4, each function will cost  2 plus itself, and additionally 2 subtracts.
- Lastly, the return costs 1.

$1 + 1 + 1 + 2 + WCET + 2 + WCET + 4 + 1 + 4 + 4 + 1$


---
2. What if the function `main()` calls `Calculate()` with value z = 5?
	```C
	int main(){
		int ans;
		ans = Calculate(5);
	}
	```
It takes 1 for `int ans` (declaration line) and 1 for `ans = ...` (assignment line).
We should have; `1 + 1 + 2 + w(5)` (ans: 188)

---
3. The deadline for this execution is 180, so determine if this execution can meet the deadline or not.?
4. For which values of addition and subtraction that the deadline should be met?

	We have the formula is `104 + 21x` for the `w(5)`, so do the Math,...

## 3.2. Old exam problem
Consider the following C program code for the function **ShifftXY**

```C
int ShiftXY(int x, int y){
	int result; // 1 unit
	int shift0; // 1 unit
	
	result = 1; //1 unit
	
	while(x != 0){ // each while is 2, plus 5 times
		xbit0 = x & 1;
		if (xbit0 == 1)
			result = result * y;
		x = x >> 1;
		y =y << 1;
	}	
	return result; // 2 units
}
```

Assume the following cost:
- declaration and assignment cost 1 unit.
- evaluation of logical condition (if, while) cost 2.
- bitwise right and left shift all cost 2 unit.
- bitwise AND costs 1
- multiply costs 5.
- return costs 2.

a. Calculate WCET following the Shaw's method. WCET is about 73 units, function is called with parameter x is in range [1, 14] and y is in range [7, 63] (range value of x and y).

Take out: to maximize iterations of while loop, the index (greatest) of set bit in x. Also maximize the if-condition by having as many set bits in x as possible.
Given x belongs to [1,14], means [0001...1110], it will take at most 4 iterations, with 5 while loops. Max population count is 3 (cannot 0 at the first bit), so we have 1110, 1101 and 1011.

Inside the while block, we have a bitwise AND, so it should be 4(assign, and, if-condition, assign, bitwise, assign, bitwise). We also have 3 (assign and multiply) (inside the if block, for 4 bits, at least 3 times, as with 3 values above).

`Total = 2*1 + 1 + 5*2 + 4*(1 + 1 + 2 + 1 + 2 + 1 + 2) + 3 * (1 + 5) + 2 = 73`

b. What is the value of x in range of [1, N] if the WCET should not exceed 67 units?
if the number of while loop iterations is 3 or less, WCET is reduced by 1 * (assign, and, compare, assign, shift, assign, shift), which equals to 10. So it always is OK to meet the deadline
if the number of loops still be 4, we should reduces the population count, since each will contribute 1 * (assign, multiply), equals to 6, so if the population is less or equal to 2, it still be ok. Hence, the bit no 3 must be set and each bits 2 to 0 should have a population count of 1, leaving only these values: 1100, 1010, 1001.
if we still use the original range, it included 1011 (not in the allowed) because it will contribute a significant value to WCET.