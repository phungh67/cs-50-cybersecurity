#data #security #pipeline

# The medallion architecture - Data pipeline

This is a framework used to manage and improve data quality through different processing language.
- Bronze: raw data
- Silver: filtered, cleaned data
- Gold: business level data

## Privacy at the Gold layer

Collecting data then used it for running analytics must be consent from customers.
- Must be consent before using (GDPR)
- Proper technical and organization measures are assumed to be in place
- Data must be locked inside the organization, used only for internal purposes.

## [[Data privacy opening|Privacy]] when [[Data privacy in sharing | sharing]] data

There are some additional constrains:
- Data maybe shared between different departments internally
- Maybe release to external organizations
- Maybe used for secondary analytics

When a data (that refined into insight), it maybe release (sharing) to enable collaboration, there are two requirements must be balance:
- Data sharing: where to share, what organizations to be shared.
- Privacy: protected individuals privacy and do not expose their personal life of situation.