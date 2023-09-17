**Task 0**: Login/Register Screen
**Description** - A single modal will switch between Login and Register Screen.
**Required Features** - Frontend & Backend Validation, Error Handling and Messages, Password Constraints, Password Match confirmation, Password Reset using Secret key auto-generated for the user.

*NOTE: For Backend Design the schema as per your need.*

Front-End Requirements -
1. Modal -
	1. Login Screen
		1. Username
		2. Password
		3. Reset Password
		4. Login Button
	2. Register Screen
		1. Complete Name
		2. Username
		3. Password
		4. Confirm Password
		5. Accept Terms and Conditions (This will enable the Register Button which is disabled by default)
		6. Register Button (On successful registration, Show one time secret)
	3. Reset Password Screen
		1. Username
		2. Reset Secret
		3. New Password
		4. Confirm New Password

Back-End Requirements - 
1. REST API-
	1.  /login
	2.  /register
	3.  /reset

Task 1: E-governance site 
Multi-Stepper forms - 
1. Personal Details 
	1. First Name
	2. Middle Name
	3. Last Name
	4. Complete Name - (AutoFilled - generated from above fields - READ ONLY)
	5. Contact
	6. DOB
	7. Age (Autofilled - Age is derived from DOB)
	8. Email
	9. Permanent Address
		1. Address Line 1
		2. Address Line 2
	10. Corresponding Address
		1. Address Line 1
		2. Address Line 2

Functionality for Frontends:
1. Sellermetrics - Currency change feature
2. Sellermetrics - Date range picker implementation

Functionality for Backends:
1. Dummy API to support Date Range Picker feature
