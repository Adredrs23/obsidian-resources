1. ADT A01 - Admit/visit notification
2. ADT A02 - Transfer a patient
3. ADT A03 - Discharge/end visit
4. ADT A04 - Register a patient
5. ADT A05 - Pre-admit a patient
6. ADT A06 - Change an outpatient to an inpatient
7. ADT A07 - Change an inpatient to an outpatient
8. ADT A08 - Update patient information
9. ADT A09 - Patient departing - tracking
10. ADT A10 - Patient arriving - tracking
11. ADT A11 - Cancel admit/visit notification
12. ADT A12 - Cancel transfer
13. ADT A13 - Cancel discharge/end visit
14. ADT A14 - Pending admit
15. ADT A15 - Pending transfer
16. ADT A16 - Pending discharge
17. ADT A17 - Swap patients
18. ADT A18 - Merge patient information
19. ADT A19 - Patient query
20. ADT A20 - Bed status update
21. ADT A21 - Patient goes on a "leave of absence"
22. ADT A22 - Patient returns from a "leave of absence"
23. ADT A23 - Delete a patient record
24. ADT A24 - Link patient information
25. ADT A25 - Cancel pending discharge
26. ADT A26 - Cancel pending transfer
27. ADT A27 - Cancel pending admit
28. ADT A28 - Add person information
29. ADT A29 - Delete person information
30. ADT A30 - Merge person information
31. ADT A31 - Update person information
32. ADT A32 - Cancel patient arriving - tracking
33. ADT A33 - Cancel patient departing - tracking
34. ADT A34 - Merge patient information - patient ID only
35. ADT A35 - Link patient information - patient ID only
36. ADT A36 - Discharge/end visit - patient not found
37. ADT A37 - Cancel discharge/end visit - patient not found
38. ADT A38 - Pending discharge - patient not found
39. ADT A39 - Cancel pending discharge - patient not found
40. ADT A40 - Generate a visit ID

| Message Type | Description                        |
|--------------|------------------------------------|
| HL7 ADT      | Admit, Discharge and Transfer      |
| HL7 ORM      | Order Entry                        |
| HL7 ORU      | Observation Result                 |
| HL7 MDM      | Medical Document Management        |
| HL7 DFT      | Detailed Financial Transactions    |
| HL7 BAR      | Billing Account Record             |
| HL7 SIU      | Scheduling Information Unsolicited |
| HL7 RDS      | Pharmacy/treatment Dispense        |
| HL7 RDE      | Pharmacy/Treatment Encoded Order   |
| HL7 ACK      | Acknowledgement Message            |

### HL7 Message Structure
**MSH**|^~\&|EPIC|EPICADT|iFW|SMSADT|199912271408|CHARRIS|ADT**^**A04|1817457|D|2.5|  
**PID**||0493575^^^2^ID 1|454721||DOE^JOHN^^^^|DOE^JOHN^^^^|19480203|M||B|254 MYSTREET AVE^^MYTOWN^OH^44123^USA||(216)123-4567|||M|NON|400003403~1129086|  
**NK1**||ROE^MARIE^^^^|SPO||(216)123-4567||EC|||||||||||||||||||||||||||  
**PV1**||O|168 ~219~C~PMA^^^^^^^^^||||277^ALLEN MYLASTNAME^BONNIE^^^^|||||||||| ||2688684|||||||||||||||||||||||||199912271408||||||002376853