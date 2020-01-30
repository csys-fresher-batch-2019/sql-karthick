# CitiPay

   ◾ CitiPay is an online service that allows an individual to make electronic transactions. An individual's bank account
     can be linked to this for enabling transactions. 
 
 ## Features
 
   ◾ Users can log in to the Citipe using their mobile numbers linked with the bank account. User's accounts cannot be 
     duplicated.They can view the current balance and transaction history.
  
 ### Feature 1 : Bank Account Database 
 
   ◾ Basic info about the bank users.
   
 ```sql
 
drop table account_details;
create table account_details(
user_name varchar2(100) not null,
mobile_no number(10),
email_id varchar2(100) not null,
account_no number(10) not null,
balance decimal(8,2) not null,
account_status varchar2(15) default 'Active',
kyc_details char(1) default 'N',
constraint user_pk unique(user_name),
constraint mob_uq primary key(mobile_no),
constraint mob_no_chk check( 6000000000<=mobile_no and mobile_no<=9999999999),
constraint email_uq unique(email_id),
constraint acc_no_pk unique(account_no),
constraint acc_no_chk check( 1000000000<=account_no and account_no<=9999999999),
constraint kyc_ck check (kyc_details in ('Y','N'))
);
insert into account_details(user_name,mobile_no,email_id,account_no,balance,kyc_details) 
values ('Karthi',6383055138,'karthick@gmail.com',5520049447,50000.00,'Y');
insert into account_details(user_name,mobile_no,email_id,account_no,balance,kyc_details)
values ('Selva',6383055145,'selva@gmail.com',5520049456,60000.00,'Y');
insert into account_details(user_name,mobile_no,email_id,account_no,balance,kyc_details) 
values ('Siva',6383055123,'siva@gmail.com',5520049347,70000.00,'Y');
insert into account_details(user_name,mobile_no,email_id,account_no,balance,account_status)
values ('Kesavan',6383054567,'kesav@gmail.com',5520049443,80000.00,'Inactive');
insert into account_details(user_name,mobile_no,email_id,account_no,balance) 
values ('Ajmal',6383567878,'ajmal@gmail.com',5520049677,90000.00);

select * from account_details;

```
| User_id | Mobile_no  |    Email_id        | Account_no | Balance | Account_status | Kyc_details |
|---------|------------|--------------------|------------|---------|----------------|-------------|
| Karthi  | 6383055138 | karthick@gmail.com | 5520049447 | 50000   | Active         | Y           |
| Selva   | 6383055145 | selva@gmail.com    | 5520049456 | 60000   | Active         | Y           |
| Siva    | 6383055123 | siva@gmail.com     | 5520049347 | 70000   | Active         | Y           |
| Ajmal   | 6383567878 | ajmal@gmail.com    | 5520049677 | 90000   | Active         | N           |
| Kesavan | 6383054567 | kesav@gmail.com    | 5520049443 | 80000   | Inactive       | N           |


 ◾ No of user accounts active 

     select count(mobile_no) from account_details where account_status='Active';

 ◾ Updating kyc_details 

     update account_details set kyc_details='Y' where user_id='Kesavan';


### Feature 2 : CitiPay login

  ◾  Users can create the login on only one condition that he/she should have a Citibank Account.
  
```sql

drop table login;
create table login(
mobile_no number(10) not null,
upi_passwd number(4) not null,
constraint mob_fk foreign key(mobile_no) references account_details(mobile_no)
);

create or replace Procedure Login_Procedure(
m_mobile_no in number,
passwd in number,
login_stats out varchar2) 
as verify_sts number;
refer number;
Begin
    verify_sts := mob_fn(m_mobile_no);
    if verify_sts=1 then
    insert into login(mobile_no,upi_passwd) values (m_mobile_no,to_char('0000'||passwd));
   login_stats := 'SUCCESS';
    commit;
    end if;
   EXCEPTION WHEN OTHERS THEN
    login_stats := 'Account doesnot exist';
End Login_Procedure;

create or replace function mob_fn (m_mob in number) 
return number as mob number:= 0;
refer number:=0;
refer1 number:=0;
Begin
   select mobile_no into refer from account_details where mobile_no=m_mob and account_status='Active;
   select count(mobile_no) into refer1 from login where mobile_no=m_mob;
   if ((refer!=m_mob) or (refer1!=0)) then
   mob:=0;
   else 
   mob:=1;
  Return mob;
  end if;
End mob_Fn;

declare
login_stats varchar2(100);
begin
login_procedure(6383055145,1001,login_stats);
dbms_output.put_line(login_stats);
end;

select * from login;

```

| Upi_passwd | Mobile_no  |
|------------|------------|
| 1001       | 6383055138 |
| 1002       | 6383055145 |
| 1003       | 6383055123 |
| 1004       | 6383567878 |


 ◾ One user cannot have multiple pins.

 ◾ Inactive accounts cannot be logged in.

 ◾ Changing the upi_passwd :

     update login set upi_passwd= 1234 where mobile_no=6383055138;


### Feature 3 : Transaction details

  ◾ Users can check the transaction history and check balance.
  
```sql

drop table transaction_table;
create table transaction_table(
mobile_no number(10) not null,
rec_mob_no number(10) not null,
categories varchar2(50),
transaction_time timestamp default sysdate,
transaction_amount number not null,
transaction_status varchar2(50) not null,
constraint amount_ck check(transaction_amount>0),
constraint status_ck check(transaction_status in ('Success','Failure')),
constraint category_ck check(categories in ('Credited','Debited','No Transaction')),
constraint mobi_fk foreign key(mobile_no) references account_details(mobile_no)
);

create or replace procedure transaction_proc(
sender_mob in number,
receiver_mob in number,
transfer_amount in number,
pin_no in number,
transaction_sts out varchar
)as confirmation1 number;
confirmation2 number;
pin_confirmation number;
final_check number;
begin
  confirmation1:=mob_fn1(sender_mob);
  confirmation2:=mob_fn1(receiver_mob);
  final_check:=status_chk(sender_mob,receiver_mob);
  select upi_passwd into pin_confirmation from login where mobile_no=sender_mob;
  if ( (confirmation1=1 and confirmation2=1) AND  transfer_amount>0 AND pin_confirmation=pin_no and final_check=1) then
      update account_details set balance = balance-transfer_amount where mobile_no=sender_mob;
      update account_details set balance = balance+transfer_amount where mobile_no=receiver_mob;
      insert into transaction_table(mobile_no,rec_mob_no,categories,transaction_amount,transaction_status) values
      (sender_mob,receiver_mob,'Debited',transfer_amount,'Success');
       transaction_sts:='Transaction Successfull';
  end if;
  commit;
  exception when others then
  transaction_sts:='Transaction Failed';
  rollback;
end transaction_proc;

create or replace function mob_fn1(m_mob in number) 
return number as mob number:= 0;
refer number:=0;
refer1 number:=0;
Begin
   select mobile_no into refer from account_details where mobile_no=m_mob;
   if (refer!=m_mob) then
   mob:=0;
   else 
   mob:=1;
  Return mob;
  end if;
End mob_fn1;

create or replace function status_chk(
s_mob in number,
r_mob in number
)return number as confirmation1 number;
confirmation2 number;
sts_chk varchar2(50);
kyc_chk char(1);
final_chk number;
begin
select account_status into sts_chk from account_details where mobile_no=s_mob;
select kyc_details into kyc_chk from account_details where mobile_no=s_mob;
if ( sts_chk='Active' and kyc_chk='Y') then
confirmation1:=1;
end if;
select account_status into sts_chk from account_details where mobile_no=r_mob;
select kyc_details into kyc_chk from account_details where mobile_no=r_mob;
if ( sts_chk='Active' and kyc_chk='Y') then
confirmation2:=1;
end if;
if(confirmation1=1 and confirmation2=1) then
final_chk:=1;
else 
final_chk:=0;
end if;
return final_chk; 
end status_chk;

declare
transaction_sts varchar2(100);
begin
transaction_proc(6383055145,6383055138,1500,1001,transaction_sts);
dbms_output.put_line(transaction_sts);
end;

select * from transaction_table;


```
◾ Successfull transactions

     select (*) from transaction_table where transaction_status='Success';
 
| Sender_mob | Receiver_mob | Category | Transaction_time               | Transaction_amount | Transaction_status |
|------------|--------------|----------|--------------------------------|--------------------|--------------------|
| 6383055138 | 6383055145   | Debited  | 03-01-20 02:00:06.000000000 AM | 1500               | Success            |
| 6383055138 | 6383055123   | Debited  | 03-01-20 02:00:56.000000000 AM | 1500               | Success            |
| 6383055145 | 6383055123   | Debited  | 03-01-20 02:01:39.000000000 AM | 1500               | Success            |
| 6383055145 | 6383055138   | Debited  | 03-01-20 02:03:14.000000000 AM | 1500               | Success            |


◾ Balance of a particular user

     select user_name,mobile_no,account_no,balance from account_details where mobile_no=6383055138;

| User_name | Mobile_no  | Account_no | Balance |
|-----------|------------|------------|---------|
| Karthi    | 6383055138 | 5520049447 | 47000   |


◾ Transaction history of a particular user

     select * from transaction_table where mobile_no=6383055138;
     
| Mobile_no  | Rec_mobile | Category | Transaction_time               | Transfer_amount | Transaction_status |
|------------|------------|----------|--------------------------------|-----------------|--------------------|
| 6383055138 | 6383055145 | Debited  | 03-01-20 02:00:06.000000000 AM | 1500            | Success            |
| 6383055138 | 6383055123 | Debited  | 03-01-20 02:00:56.000000000 AM | 1500            | Success            |
| 6383055138 | 6383055145 | Debited  | 03-01-20 02:11:41.000000000 AM | 1500            | Success            |  











## table 1

create table account_details(
user_name varchar2(100) not null,
mobile_no number(10),
email_id varchar2(100) not null,
account_no number(10) not null,
balance decimal(18,2) not null,
account_status varchar2(15) default 'Active',
constraint user_pk unique(user_name),
constraint mob_uq primary key(mobile_no),
constraint mob_no_chk check( 6000000000<=mobile_no and mobile_no<=9999999999),
constraint email_uq unique(email_id),
constraint acc_no_pk unique(account_no),
constraint acc_no_chk check( 1000000000<=account_no and account_no<=9999999999)
);
insert into account_details(user_name,mobile_no,email_id,account_no,balance) values ('Karthi',6789012340,'karthick@gmail.com',5000000001,100000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,balance) values ('Selva',6789012341,'selva@gmail.com',5000000002,100000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,balance) values ('Siva',6789012342,'siva@gmail.com',5000000003,100000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,balance) values ('Kesavan',6789012343,'kesav@gmail.com',5000000004,100000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,balance,account_status) values ('Ajmal',6789012344,'ajmal@gmail.com',5000000005,100000.00,'Inactive');
insert into account_details(user_name,mobile_no,email_id,account_no,balance) values ('Manoj',6789012345,'manoj@gmail.com',5000000006,100000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,balance) values ('Vijay',6789012346,'vijay@gmail.com',5000000007,100000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,balance) values ('Aravinth',6789012347,'aravinth@gmail.com',5000000008,100000.00);
insert into account_details(user_name,mobile_no,email_id,account_no,balance) values ('Naresh',9999999999,'naresh@gmail.com',5000000009,100000.00);

select * from account_details;

## table 2

create table login(
mobile_no number(10) not null,
upi_passwd number(4) not null,
constraint upi_pass_chk check(1000<=upi_passwd and upi_passwd<=9999),
constraint mob_fk foreign key(mobile_no) references account_details(mobile_no)
);

select * from login;

## table 3


create table transaction_table(
mobile_no number(10) not null,
rec_mob_no number(10) not null,
categories varchar2(50),
transaction_time timestamp default sysdate,
transaction_amount number not null,
transaction_status varchar2(50) default 'Success',
constraint amount_ck check(transaction_amount>0),
constraint category_ck check(categories in ('Credited','Debited','Failed')),
constraint transaction_chk check(transaction_status in ('Success','Failure')),
constraint mobi_fk foreign key(mobile_no) references account_details(mobile_no)
);


## table 4

create table kyc(
mobile_no number(10) not null,
aadhar_no varchar(12),
kyc_username varchar2(50),
passport_no varchar2(8),
driving_license_no varchar2(15),
kyc_status varchar2(5) default 'N',
kyc_wallet number,
constraint aadhar_no_uq unique(aadhar_no),
constraint passport_no_uq unique(passport_no),
constraint driving_no_uq unique(driving_license_no),
--constraint aadhar_chk check(aadhar_no>200000000000 and aadhar_no<999999999999),
constraint mobile_for_k foreign key(mobile_no) references account_details(mobile_no),
constraint kyc_chk check (kyc_status in ('Y','N'))
);

## table 5 -- not implemented

create table merchant_details(
merchant_id number not null,
company_name varchar2(100) not null,
api_key number,
account_no number(10) not null,
merchant_status varchar2(1) default 'N',
mobile_no number(10) not null,
constraint mob_no_ck check( 6000000000<=mobile_no and mobile_no<=9999999999),
constraint acc_no unique(account_no),
constraint acc_ck check( 1000000000<=account_no and account_no<=9999999999),
constraint mob_fr_k foreign key(mobile_no) references account_details(mobile_no)
);


### Procedures


## procedure declaration for login

create or replace Procedure Login_Procedure(
m_mobile_no in number,
passwd in number,
login_stats out varchar2) 
as verify_sts number;
refer number;
passchk number;
refer1 number:=0;
Begin
    verify_sts := mob_fn(m_mobile_no);
    if verify_sts=1 then
        select count(mobile_no) into refer1 from login where mobile_no=m_mobile_no;
            if(refer1=0) then
                insert into login(mobile_no,upi_passwd) values (m_mobile_no,passwd);
                login_stats := 'Account created';
                commit;
            else
                select upi_passwd into passchk from login where mobile_no = m_mobile_no;
                if(passchk=passwd) then
                     login_stats := 'Account logged-in';
                else
                    login_stats := 'Incorrect username/password';
                end if;
             end if;
    else
    login_stats := 'Account doesnot exist';
    end if;
EXCEPTION WHEN OTHERS THEN
login_stats := 'Account doesnot exist';
End Login_Procedure;

## procedure call for login
declare
login_stats varchar2(100);
begin
login_procedure(9999999999,1234,login_stats);
dbms_output.put_line(login_stats);
end;


## procedure declaration for verifying mobile number

create or replace procedure mob_chk_proc(
mob_no in number,
mob_verification out number
) as 
verification number:=0;
begin
    select count(mobile_no) into verification from login where mobile_no=mob_no;
    if(verification!=0) then
        mob_verification:=0;
    else
        mob_verification:=1;
     end if;   
end mob_chk_proc;

## procedure declaration for bank transaction 

create or replace procedure transaction_proc(
sender_mob in number,
receiver_mob in number,
transfer_amount in number,
pin_no in number,
transaction_sts out varchar
)as confirmation1 number;
confirmation2 number;
pin_confirmation number;
final_check number;
transfer_chk decimal;
begin
  --confirmation1:=mob_fn1(sender_mob);
  confirmation2:=mob_fn1(receiver_mob);
  final_check:=status_chk(receiver_mob);
  --select upi_passwd into pin_confirmation from login where mobile_no=sender_mob;
  select balance into transfer_chk from account_details where mobile_no=sender_mob;
  if ( (confirmation2=1) AND  transfer_amount>0 and final_check=1) then
    if(transfer_chk>=transfer_amount) then
      update account_details set balance = balance-transfer_amount where mobile_no=sender_mob;
      update account_details set balance = balance+transfer_amount where mobile_no=receiver_mob;
      insert into transaction_table(mobile_no,rec_mob_no,categories,transaction_amount,transaction_status) values
      (sender_mob,receiver_mob,'Debited',transfer_amount,'Success');
      insert into transaction_table(mobile_no,rec_mob_no,categories,transaction_amount,transaction_status) values
      (receiver_mob,sender_mob,'Credited',transfer_amount,'Success');
       transaction_sts:='Transaction Successfull';
        commit;
    else
        transaction_sts:='Not sufficient balance';
        end if;
  else
  transaction_sts:='Transaction Failed';
        rollback;
  insert into transaction_table(mobile_no,rec_mob_no,categories,transaction_amount,transaction_status) values (sender_mob,receiver_mob,'Failed',transfer_amount,'Failure');
   commit;
   end if;
end transaction_proc;

## procedure call for transaction

declare
transaction_sts varchar2(100);
begin
transaction_proc(6789012341,6789012345,5000,1001,transaction_sts);
dbms_output.put_line(transaction_sts);
end;


## procedure declaration for wallet transaction 

create or replace procedure wallet_proc(
sender_mob in number,
receiver_mob in number,
transfer_amount in number,
transaction_sts out varchar
) as 
confirmation number;
final_check number;
balance_amnt number;
begin
    confirmation:=mob_fn1(receiver_mob);
    final_check:=status_chk(receiver_mob);
    select kyc_wallet into balance_amnt from kyc where mobile_no=sender_mob;
    if ((confirmation=1) and transfer_amount>0 and final_check=1) then
        if(balance_amnt>=transfer_amount) then
            update kyc set kyc_wallet = kyc_wallet-transfer_amount where mobile_no=sender_mob;
            update account_details set balance = balance+transfer_amount where mobile_no=receiver_mob;
            --update wallet_details set balance = balance+transfer_amount where mobile_no=receiver_mob;
            insert into transaction_table(mobile_no,rec_mob_no,categories,transaction_amount,transaction_status) values
                (sender_mob,receiver_mob,'Debited',transfer_amount,'Success');
            insert into transaction_table(mobile_no,rec_mob_no,categories,transaction_amount,transaction_status) values
                (receiver_mob,sender_mob,'Credited',transfer_amount,'Success');
            transaction_sts:='Transaction Successfull';
            commit;
        else
            transaction_sts:='Not sufficient balance';
            end if;
  else  
    transaction_sts:='Transaction Failed';
    rollback;
    insert into transaction_table(mobile_no,rec_mob_no,categories,transaction_amount,transaction_status) values (sender_mob,receiver_mob,'Failed',transfer_amount,'Failure');
    commit;
end if;
end wallet_proc;

## procedure call for wallet transaction 

declare
transaction_sts varchar2(100);
begin
wallet_proc(6789012340,6789012345,50,transaction_sts);
dbms_output.put_line(transaction_sts);
end;



## procedure declaration for kyc status check

create or replace Procedure Kyc_Status_Chk_Proc(
Mob_No In Number,
Verified_Result Out Number
)
As 
kyc_chk char(1);
refer number;
Begin
    select count(mobile_no) into refer from kyc where mobile_no=Mob_No;
    if(refer!=0) then
        select kyc_status into kyc_chk from kyc where mobile_no=Mob_No;
        if(kyc_chk='Y') then
            verified_result:=1;
        else
            verified_result:=0;
        end if;
    else
        verified_result:=0;
    end if;
End Kyc_Status_Chk_Proc;

## procedure declaration for adding kyc details

create or replace procedure KYC_PROC(
kyc_mobile in number,
kyc_aadhar in varchar,
kyc_passport in varchar,
kyc_driving in varchar,
kycc_username in varchar,
kyc_sts out varchar)
as
count_verification number;
begin
    select count(mobile_no) into count_verification from kyc where mobile_no=kyc_mobile;
        if((kyc_aadhar is not null) and (kyc_passport is null) and (kyc_driving is null)) then
                if(count_verification=0) then
                    insert into kyc(mobile_no,aadhar_no,kyc_username,kyc_status,kyc_wallet) values (kyc_mobile,kyc_aadhar,kycc_username,'Y',50);
                    commit;
                    kyc_sts:='Kyc updated';
                else
                    kyc_sts:='Kyc already exists.Cannot be updated';
                    end if;
        elsif((kyc_aadhar is null) and (kyc_passport is not null) and (kyc_driving is null)) then
                if(count_verification=0) then
                    insert into kyc(mobile_no,passport_no,kyc_username,kyc_status,kyc_wallet) values (kyc_mobile,kyc_passport,kycc_username,'Y',50);
                    commit;
                    kyc_sts:='Kyc updated';
                else
                    kyc_sts:='Kyc already exists.Cannot be updated';
                    end if;
        elsif((kyc_aadhar is null) and (kyc_passport is null) and (kyc_driving is not null)) then
            if(count_verification=0) then
                    insert into kyc(mobile_no,driving_license_no,kyc_username,kyc_status,kyc_wallet) values (kyc_mobile,kyc_driving,kycc_username,'Y',50);
                    commit;
                    kyc_sts:='Kyc updated';
            else
                    kyc_sts:='Kyc already exists.Cannot be updated';
                    end if;
        else
        kyc_sts:='Invalid input';
            end if;
end KYC_PROC;

## procedure call for adding kyc details


declare
kyc_sts varchar2(100);
begin
kyc_proc(9999999999,'777788889999',null,null,'Naresh',kyc_sts);
end;

### Functions

## function declaration for checking their status(Active/Inactive)

create or replace function mob_fn (m_mob in number) 
return number as mob number:= 0;
refer number:=0;
Begin
   select mobile_no into refer from account_details where mobile_no=m_mob and account_status='Active';
   if (refer=0) then
   mob:=0;
   else 
   mob:=1;
  Return mob;
  end if;
End mob_Fn;

select mob_fn(6383054567) from dual;


## function declaration for mobile verification

create or replace function mob_fn1(m_mob in number) 
return number as mob number:= 0;
refer number:=0;
refer1 number:=0;
Begin
   select mobile_no into refer from account_details where mobile_no=m_mob;
   if (refer!=m_mob) then
   mob:=0;
   else 
   mob:=1;
  Return mob;
  end if;
End mob_fn1;

select mob_fn1(6383054567) from dual;


## function declaration for kyc status check

create or replace function status_chk(
r_mob in number
)return number as confirmation1 number;
confirmation2 number;
sts_chk2 varchar2(50);
final_chk number;
begin
select account_status into sts_chk2 from account_details where mobile_no=r_mob;
if ( sts_chk2='Active') then
confirmation2:=1;
end if;
if(confirmation2=1) then
final_chk:=1;
else 
final_chk:=0;
end if;
return final_chk; 
end status_chk;

select status_chk(6383055138,6383055145) from dual;




