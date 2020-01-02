# CitiPe
   ◾ CitiPe is an online service that allows an individual to make electronic transactions. An individual's bank account
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

### Feature 2 : CitiPe login
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
sts_chk1 varchar2(50);
kyc_chk1 char(1);
sts_chk2 varchar2(50);
kyc_chk2 char(1);
final_chk number;
begin
select account_status into sts_chk1 from account_details where mobile_no=s_mob;
select kyc_details into kyc_chk1 from account_details where mobile_no=s_mob;
if ( sts_chk1='Active' and kyc_chk1='Y') then
confirmation1:=1;
end if;
select account_status into sts_chk2 from account_details where mobile_no=r_mob;
select kyc_details into kyc_chk2 from account_details where mobile_no=r_mob;
if ( sts_chk2='Active' and kyc_chk2='Y') then
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
 
 
| Sender_mob | Receiver_mob | Category | Transaction_time               | Transaction_amount | Transaction_status |
|------------|--------------|----------|--------------------------------|--------------------|--------------------|
| 6383055138 | 6383055145   | Debited  | 03-01-20 02:00:06.000000000 AM | 1500               | Success            |
| 6383055138 | 6383055123   | Debited  | 03-01-20 02:00:56.000000000 AM | 1500               | Success            |
| 6383055145 | 6383055123   | Debited  | 03-01-20 02:01:39.000000000 AM | 1500               | Success            |
| 6383055145 | 6383055138   | Debited  | 03-01-20 02:03:14.000000000 AM | 1500               | Success            |


◾ Balance after transaction 

| User_name | Mobile_no  | Email_id           | Account_no | Balance | Account_status | Kyc_details |
|-----------|------------|--------------------|------------|---------|----------------|-------------|
| Karthi    | 6383055138 | karthick@gmail.com | 5520049447 | 47000   | Active         | Y           |
| Selva     | 6383055145 | selva@gmail.com    | 5520049456 | 60000   | Active         | Y           |
| Siva      | 6383055123 | siva@gmail.com     | 5520049347 | 73000   | Active         | Y           |
| Kesavan   | 6383054567 | kesav@gmail.com    | 5520049443 | 80000   | Inactive       | N           |
| Ajmal     | 6383567878 | ajmal@gmail.com    | 5520049677 | 90000   | Active         | N           |
