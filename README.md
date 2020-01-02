# CitiPe
   ◾ CitiPe is an online service that allows an individual to make electronic transactions. An individual's bank account
     can be linked to this for enabling transactions. 
 
 ## Features
   ◾ Users can log in to the Citipe using their mobile numbers linked with the bank account. User's accounts cannot be 
     duplicated.They can view the current balance and transaction history.
  
 ### Feature 1 : Bank Account Database 
   ◾ Linking the Users can create the login on only one condition that he/she should have a Citibank Account.
 
 ```sql
 
create table account_details(
user_name varchar2(100),
mobile_no number(10) not null,
email_id varchar2(100) not null,
account_no number(10) not null,
account_type varchar2(10) not null,
balance decimal(8,2) not null,
account_status varchar2(15) default 'Active',
constraint user_pk primary key(user_name),
constraint mob_uq unique(mobile_no),
constraint mob_no_chk check( 6000000000<=mobile_no and mobile_no<=9999999999),
constraint email_uq unique(email_id),
constraint acc_no_pk unique(account_no),
constraint acc_no_chk check( 1000000000<=account_no and account_no<=9999999999),
constraint acc_type_chk check(account_type in ('Savings','Current'))
);
insert into account_details(user_name,mobile_no,email_id,account_no,account_type,balance,account_status) values ('Karthi',6383055138,'karrthicks10@gmail.com',5520049447,'Savings',50000.00,'Inactive');
insert into account_details(user_name,mobile_no,email_id,account_no,account_type,balance) values ('Selva',6383055145,'selva10@gmail.com',5520049456,'Savings',60000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,account_type,balance) values ('Siva',6383055123,'siva@gmail.com',5520049347,'Savings',70000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,account_type,balance,account_status) values ('Kesavan',6383054567,'kesav@gmail.com',5520049443,'Current',80000.00,'Inactive');
insert into account_details(user_name,mobile_no,email_id,account_no,account_type,balance) values ('Ajmal',6383567878,'ajmal@gmail.com',5520049677,'Current',90000.00);

select * from account_details;

```
| User_id | Mobile_no  |    Email_id        | Account_no | Balance | Account_status | Kyc_details |
|---------|------------|--------------------|------------|---------|----------------|-------------|
| Karthi  | 6383055138 | karthick@gmail.com | 5520049447 | 50000   | Active         | Y           |
| Selva   | 6383055145 | selva@gmail.com    | 5520049456 | 60000   | Active         | Y           |
| Siva    | 6383055123 | siva@gmail.com     | 5520049347 | 70000   | Active         | Y           |
| Ajmal   | 6383567878 | ajmal@gmail.com    | 5520049677 | 90000   | Active         | N           |
| Kesavan | 6383054567 | kesav@gmail.com    | 5520049443 | 80000   | Inactive       | N           |

 
