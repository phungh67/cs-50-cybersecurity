#data #privacy

# De-identification: Data focus

> [!Definition]
> De-identification pertains to separating an individual's identity from a specific dataset. This protects privacy with still allowing for data analysis and sharing without compromising the confidentiality of the individuals question.

## Understand attributes

We mostly refer to tables (in the database) when considering about data privacy. We define a table T with columns name A1, A2,... as attributes of T.

## Personal identifiable information (PII)

>[! Definition]
> PII information that can be used to distinguish or trace an individual’s identity, either alone or when combined with other information that is linked or linkable to a specific individual.

There are two ways to obtain:
- Directly identifiers:
	- Full name
	- Passport number
	- Social security number
	- Driving license number
- Indirect:
	- ZIP code
	- Birthdate
There is also a term called "Power of combination". Take an example from America, while just using ZIP code, birthdate and gender, with a help from voters list, 87% American identities are exposable.

## Remove PII

Anonymization is a de-identification technique. In its most elemental form, it removes the information about PIIs from datasets.
- Remove PII
- Mask PII
- Pseudonymization for PII

## The probability of re-identification after Anonymization
