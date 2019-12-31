# Citi-Wallet
   ◾ Citi-Wallet is an online service that allows an individual to make electronic transactions. An individual's bank account
     can be linked to this wallet for enabling transactions. Users can also store their driving license, pan card, voters id in it.
 
 ## Features
   ◾ Users can log in to the Citi-Wallet using their mobile numbers linked with the bank account. User's accounts cannot be 
     duplicated.They can view the current balance and transaction history. They can view the stored ID information in the wallet.
  
 ### Feature 1 : Bank Account creation 
   ◾ Users can create the login on only one condition that he/she should have a Citibank Account. If the user doesn't have a
     Citibank account, he/she should create a bank account directly contacting the bank. 
 
 ```sql
 
create table account_details(
user_name varchar2(100) not null,
mobile_no number(10) not null,
email_id varchar2(100) not null,
account_no number(10),
account_type varchar2(10) not null,
balance decimal(8,2) not null,
account_status varchar2(15) default 'Active',
constraint user_uq unique(user_name),
constraint mob_no_chk check( 6000000000<=mobile_no and mobile_no<=9999999999),
constraint mob_uq unique(mobile_no),
constraint email_uq unique(email_id),
constraint acc_no_pk primary key (account_no),
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

 
