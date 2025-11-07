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
- Remove PII: this simply removes the data that are sensitive according to the ISO standard about the "directed identification information". The removal of these information provides an extra protection layer for the data, but makes additional difficulties when reassembling data.
- Mask PII: turns these information into asterisk "\*" or "X".
- Pseudonymization for PII: turns information in to some sort of "pseudonyms". For example, the records with the same characteristics (sharing a same city, same district) will be turned to another (Person A and B, they both live in Boston, a real palce -> they live in DreamCast, both live in there). This allows data to be covered and later recovered by a unique mechanism or algorithm.

## The probability of re-identification after Anonymization

Data can be re-identified by linkage attack, that cross reference between tables to find the possibility to deanonymize sensitive data.
A report that 87% of Americans can be identified with just their ZIP code, birthdate and sex.
## The attack process
- Identify unique record, link these information to obtain the anonymized data.
- To be familiar with this type of attack, there are two definition:
	- Sensitive dataset: contains the data that exclusive to an entity, for example, a social security number, name, birthdate, address,...
	- Disclosed set: only contains information that are consented to be share (under GDPR).

## Quasi-identification: a challenge for the de-identified process.

With the sensitive PII information, the best way to handle them is stripped out from the dataset because the later analytical work can be done without these things (you still can predict the "trend" for buying some sort of products in a range of "age", not about the people living in the same neighborhood). But with the **Quasi-identified** information, it is another story because these information, let alone, you can not extract some meaningful data about the identity of that record, but these information is valuable to the analysis process. Some examples are: ZIP code - you cannot know the exact location of a user, they can move anytime! or sex, although there are many pronouns, we just have 2 sexes in the medical record.

With these challenges, fortunately, we have some method to ensure the privacy:
- Suppression: completely remove that record, the privacy can be ensure but the data utilization is not (because data is lost).
- Generalization: change these data into a range instead of an exact value. For example, with zip code 12345 and 12333, we can write them as 12300-12400.
- Perturbation: specific value can be replaced with other values in an uniform manner, for example, all ages can be randomly adjusted -/+ 2 years.
- Swapping: **quasi-identification** values can be exchanged between records.
- Sub-sampling: Instead of releasing an entire dataset, the organization can release just a sample.

![[Linkage attack.png]]
In the above figure, a sensitive record *s*  must be protected to ensure the privacy. But the linkage attack can exploit this information by cross-referencing. Let's take a record *i'* from the identification dataset that might be bought (like the voting information during some election?) then combining it with a record *d* of the disclosed set (the original set with all sensitive information hidden) (and this set can be release in the internet, a prize from some hackathon,...). If the record *s* is uniquely enough, the sensitive information can be exposed [Example](https://nvlpubs.nist.gov/nistpubs/ir/2015/NIST.IR.8053.pdf)

## Measuring the threat of re-identified
[Article](https://bmcmedinformdecismak.biomedcentral.com/articles/10.1186/1472-6947-12-66)
As described earlier, an attacker can reassembly the data of a person if he/she got both disclosure set (pretty easy) and the identification set (can be purchased).
![[Threat of re-identification.png]]
As in the illustration above, there are 9 unique records (highlighted) in the disclosure set and we also have 6 unique records in the ID set. The linkage attack will scan the disclosure and the ID, performed some matching function and eventually exposed the identity of a voter (e.g Alan Smith, a male birth in 1962, will likely have test result -ve). The purpose of linkage is not to expose someone identity but to embarrass and publish the medical record (which contains sensitive information about health status as well as disease). As in this example, if the uniqueness of the record "Alan Smith" is standout (there is only 1 record matched in the disclosure set), the attack was successfully exposed his medical status (-ve).

In order to do that, we have some formular to calculate the probability of the successfully linkage attack with the quasi-identifier.

Let $N$ and $n$ be the number of records in the voter registration list and the disclosed (sample) data set respectively, $K$ and $u$ denote the number of non-zero equivalence classes in the voter registration list and the disclosed data set respectively, and $F_i$ and $f_i$ denote the size of the $i^{th}$ equivalence class in the voter registration list and the disclosed data set respectively, where $i \in \{1, \dots, K\} \text{ (or } \{1, \dots, u\} \text{ respectively)}$. ## Measuring uniqueness One can measure the conditional probability that a record in the voter registration list is unique given that it is unique in the original data set by: 
$$ \lambda_1 = \frac{\sum_i I(f_i=1, F_i=1)}{\sum_i I(f_i=1)} \quad (1) $$
where $I$ is the indicator function. For example, $I(f_i=1, F_i=1)=1$ is one if the sample equivalence class is a unique as well as the corresponding population equivalence class, otherwise it is zero. 
However, as a risk metric for the whole data set that will be disclosed, $\lambda_1$ can be misleading. In our example, 2 out of 9 sample unique records were population unique, giving a risk of $\lambda_1 = 0.22$. However, out of the whole data set only 2 out of 14 records are at risk, therefore the data set risk should be 0.14. To give a more extreme example, consider a 1000 record data set where there are only two unique records and they are both also unique in the voter registration list. In this case $\lambda_1=1$ indicating that *all* records are at risk, when in fact only 2 out of 1000 records are at risk. A more appropriate risk metric would then be: $$ \lambda_2 = \frac{\sum_i I(f_i=1, F_i=1)}{n} \quad (2) $$
In the 1000 record example above, this would give a risk of $\lambda_2 = 0.002$ and for the example of Figure 1 it would be $\lambda_2 = 0.14$ for the original data set, which corresponds to what one would expect intuitively. The risk metric $\lambda_2$ approximates the proportion of records in the voter registration list that are unique under an assumption of sampling with equal probabilities [54]. The $\lambda_3$ measure is the proportion of records in the voter registration list that are unique: 
$$ \lambda_3 = \frac{\sum_i I(F_i = 1)}{N} \quad (3) $$The value for $\lambda_3$ in our example of Figure 1 would be 0.15 since six records in the voter registration list are unique. [Source](https://bmcmedinformdecismak.biomedcentral.com/articles/10.1186/1472-6947-12-66)

## Limitations
- All these theories are just statical assumption, therefore can be changed under different conditions: different disclosure set, different quasi-identifier,...
- As stated before, choosing a good quasi-identifier set to measure the risk is very hard.
![[The variation of quasi-identifier.png]]
