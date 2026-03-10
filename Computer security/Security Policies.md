
We have several concepts:
- Security policy
- Security enforcement
- Security breach
- Security model

We also have something called "Military security model":

>[!Example]
>We have 2 objects, S and O, which S is statement/Someone and O is an object. S has right to access O if only rank S is higher than rank O and the data S needs to know is a compartment of O

**Carl-Wilson**: try to form a framework between internal data and the external which is outside of the "area" that these data refers to (for example, A is needed for B, but B is outside of the mother organization owns A, so what is the relation between these two).
-> An addition that involves *separation of duty*: the aim is to prevent abuse that can arise when the same person performs too many related actions in a company.

**Chinese wall policy** (keyword, important, need to learn or gaining additional information)
"A person can access info as long as that person has never accessed info from a different company in the same CC"


# 1. Definition

>[!Security policy]
>A statement that partitions the states of a system into a set of authorized, or secure state and a set of unauthorized, or non secure states.

So we have **a secure system** is a system that unauthorized state cannot enter it. Following, a **security breach** occurs when a system enters an unauthorized state. 
Along with that, we have **security mechanism** is an entity or procedure that enforces some part of the security policy.
And for academic context, we have **security model** - represent a particular set of security policies.

In this concept, we have two main branches:
- *Military security* focuses on Confidentiality, only authorized personnel can access and even the actions that person can perform on each data set are limited. This model has 5 levels: unclassified, restricted, confidential, secret and top secret. Along with it, information are stored in some manner that "dots-relate-to-dots" into compartments. The combination `<rank; compartment>` indicates that the highest level of security a person can perform in a scope (of information). Any person only has privilege to object that satisfies both conditions: that object is less or equal than user's rank `AND` the necessary information belongs to that object.
- *Commercial security* has a much large scope than the military one and it aims to solve the Integrity and Availability. So that there is no formal rule on the accessibility of these information. Because of that, several pioneers have established some model for this case: Clark-Wilson, Lee-Nash-Poland, Chinese Wall, and Bell-La Padula 

# 2. Clark-Wilson model

Aim to preserve "Integrity", defines a **well-formed** transaction behavior. This model is widely applied in the logistic process. 

A **well-formed transaction** is a transaction that:
- Order important: must have a copy before processing any transaction.
- No transaction is executed unless approved by authorized individual.

But this model also raises a concern that: if the super user gone wrong?

So we have **Lee-Nash-Poland** suggested an addition to the existing model that, "separation of duty" to avoid the scenario that a person can perform to many actions in a company (and of course, these actions must be related).

# 3. Chinese Wall

>[! Definition]
>Prevent flow of information between companies that may have conflicting interests (i.e same industrial, same products,...). And also, the same employee may not access information from different companies in the same conflict class. This address the "confidentiality".

# 4. BLP

Bell-La Padula enforces the information flow or precisely, the allowable paths of information flow in a secure system (confidentiality).

No read up, No write down: which is, a person can only read the data at the same level (or less than current permission) and cannot write down to that low level, to prevent unauthorized data modification.

But this model has some disadvantages:
- Higher users cannot talk with low level (read only).
- Every one can create garbage information in the higher level.

So we have *Biba model*
