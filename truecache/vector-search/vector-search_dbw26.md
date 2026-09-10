# Semantic Cache Using Vector Search

## Introduction

This lab adds semantic retrieval to the existing TRANSACTIONS payment workflow using Oracle AI Vector Search and Oracle True Cache. Semantic caching allows semantically similar requests to be served directly from True Cache, reducing repeated vector searches, LLM calls and token usage, which helps lower latency and AI costs while offloading the primary database.

The vector table is built from PAYMENTS, keeping the search results aligned with the existing schema. The environment includes a 20,000-row PAYMENT_VECTORS sample, and the lab demonstrates how to create the table, generate embeddings, build the vector index, and run similarity searches against True Cache.

![Full LiveLab semantic cache using vector search](images/full-livelab-vector-search.png " ")

Estimated Time: 15 minutes.

## Objectives

- Create a native Oracle vector table in the TRANSACTIONS schema.
- Generate 16-dimension payment embeddings from existing payment attributes.
- Create a cosine IVF vector index.
- Run similar-payment, account-behavior, and cross-border queries through True Cache.
- Understand why the result is the top five rows and how to read cosine distance.

## Task 1: Semantic Cache Using Vector Search

Run the following commands in the host terminal. The database commands use SYSDBA authentication inside the database containers, so no database password is placed in a command or displayed on screen.

Open Primary SQL*Plus:

~~~text
<copy>
sudo podman exec -it prod /bin/bash
export ORACLE_SID=ORCLCDB
sqlplus / as sysdba
alter session set container=ORCLPDB1;
</copy>
~~~

Review the payment attributes that will be represented in the vector:

~~~text
<copy>
select id, account_id, country_cd, amount, created_utc
from transactions.payments
fetch first 10 rows only;
</copy>
~~~

Create the native vector table. The PL/SQL block leaves an existing table in place and creates it when the lab is initialized from an empty schema:

~~~text
<copy>
begin
  execute immediate 'create table TRANSACTIONS.PAYMENT_VECTORS (payment_id number primary key, account_id number, country_cd varchar2(8), amount number, created_utc timestamp, embedding vector(16, float32))';
exception
  when others then
    if sqlcode != -955 then raise; end if;
end;
/
</copy>
~~~

Generate the 16-dimension embedding from the existing payment fields. The first dimensions represent amount, account, country, and transaction time. The stable hash dimensions distinguish otherwise similar payment rows:

~~~text
<copy>
merge into TRANSACTIONS.PAYMENT_VECTORS target
using (
  select id, account_id, country_cd, amount, created_utc,
    to_vector('[' ||
      to_char(least(greatest(amount,0)/1000,1),'FM0.000') || ',' ||
      to_char(mod(account_id,1000)/1000,'FM0.000') || ',' ||
      to_char(ascii(substr(country_cd,1,1))/255,'FM0.000') || ',' ||
      to_char(ascii(substr(country_cd,2,1))/255,'FM0.000') || ',' ||
      to_char(extract(month from created_utc)/12,'FM0.000') || ',' ||
      to_char(extract(day from created_utc)/31,'FM0.000') || ',' ||
      to_char(extract(hour from created_utc)/24,'FM0.000') || ',' ||
      to_char(mod(id,997)/997,'FM0.000') || ',' ||
      to_char(mod(account_id,97)/97,'FM0.000') || ',' ||
      to_char(mod(id,89)/89,'FM0.000') ||
      ',0.100,0.080,0.060,0.050,0.040,0.030]') embedding
  from TRANSACTIONS.PAYMENTS
  where rownum <= 20000
) source
on (target.payment_id = source.id)
when not matched then insert
  (payment_id, account_id, country_cd, amount, created_utc, embedding)
  values
  (source.id, source.account_id, source.country_cd, source.amount, source.created_utc, source.embedding);
commit;
select count(*) vector_rows from TRANSACTIONS.PAYMENT_VECTORS;
</copy>
~~~

Expected result: the count is 20,000 in the pre-provisioned sample. If you initialized the table yourself, the count reflects the rows available in `PAYMENTS` up to the 20,000-row sample limit.

Create the cosine IVF vector index, or rebuild it if the table was refreshed. Truncating an IVF base table marks its vector index unusable, so the existing-index branch must rebuild it before searching:

~~~text
<copy>
declare
  v_index_count number;
begin
  select count(*)
    into v_index_count
    from dba_indexes
   where owner = 'TRANSACTIONS'
     and index_name = 'PAYMENT_VECTORS_IVF_IDX';
  if v_index_count = 0 then
    execute immediate 'create vector index TRANSACTIONS.PAYMENT_VECTORS_IVF_IDX on TRANSACTIONS.PAYMENT_VECTORS (embedding) organization neighbor partitions distance cosine with target accuracy 90';
  else
    execute immediate 'alter index TRANSACTIONS.PAYMENT_VECTORS_IVF_IDX rebuild online';
  end if;
end;
/
select index_name, index_type, status
from dba_indexes
where owner = 'TRANSACTIONS'
  and index_name = 'PAYMENT_VECTORS_IVF_IDX';
</copy>
~~~

Expected result: `PAYMENT_VECTORS_IVF_IDX` is listed with a vector index type and status **VALID**.

Select a reference payment:

~~~text
<copy>
select payment_id, account_id, country_cd, amount
from TRANSACTIONS.PAYMENT_VECTORS
fetch first 1 row only;
</copy>
~~~

Leave the Primary SQL*Plus session and container:

~~~text
<copy>
exit
exit
</copy>
~~~

Run the nearest-neighbor query through True Cache. Replace 1 with the payment ID returned above:

~~~text
<copy>
sudo podman exec -it truedb /bin/bash
export ORACLE_SID=TRUEDB
sqlplus / as sysdba
set pages 100 lines 220
alter session set container=ORCLPDB1;
select payment_id, account_id, country_cd, amount,
       round(vector_distance(embedding,
         (select embedding from TRANSACTIONS.PAYMENT_VECTORS where payment_id=1),
         cosine), 6) distance
from TRANSACTIONS.PAYMENT_VECTORS
where payment_id <> 1
order by vector_distance(embedding,
  (select embedding from TRANSACTIONS.PAYMENT_VECTORS where payment_id=1), cosine)
fetch first 5 rows only;
</copy>
~~~

This is a nearest-neighbor query. It compares each payment embedding with the selected payment embedding, sorts by cosine distance, and returns the top five rows. A smaller distance means that the numeric profiles are more similar.

Run the account behavior query:

~~~text
<copy>
set pages 100 lines 220
select payment_id, account_id, country_cd, amount,
       round(vector_distance(embedding,
         (select embedding from TRANSACTIONS.PAYMENT_VECTORS where payment_id=1),
         cosine), 6) distance
from TRANSACTIONS.PAYMENT_VECTORS
where account_id = (select account_id from TRANSACTIONS.PAYMENT_VECTORS where payment_id=1)
  and payment_id <> 1
order by vector_distance(embedding,
  (select embedding from TRANSACTIONS.PAYMENT_VECTORS where payment_id=1), cosine)
fetch first 5 rows only;
</copy>
~~~

This query narrows the candidate set to the selected account and then ranks its payments by vector similarity. It demonstrates account behavior without changing the source PAYMENTS table.

Run the cross-border risk query:

~~~text
<copy>
set pages 100 lines 220
select payment_id, account_id, country_cd, amount,
       round(vector_distance(embedding,
         (select embedding from TRANSACTIONS.PAYMENT_VECTORS where payment_id=1),
         cosine), 6) distance
from TRANSACTIONS.PAYMENT_VECTORS
where country_cd <> (select country_cd from TRANSACTIONS.PAYMENT_VECTORS where payment_id=1)
order by vector_distance(embedding,
  (select embedding from TRANSACTIONS.PAYMENT_VECTORS where payment_id=1), cosine)
fetch first 5 rows only;
exit
exit
</copy>
~~~

This query looks for similar payment profiles from another country. The country filter provides the investigation scope and the vector distance provides the ranking inside that scope.

## Completion

The lab is complete when:

- PAYMENT_VECTORS contains the payment embedding sample.
- PAYMENT_VECTORS_IVF_IDX is present and **VALID**.
- The similar-payment query returns five rows through True Cache.
- The account behavior query returns the closest rows for the selected account.
- The cross-border query returns the closest rows from another country.
- The SQL and distance value are understood for each search.

## Acknowledgements

* **Authors** - Sambit Panda, Consulting Member of Technical Staff, Oracle Database Product Management
* **Contributors** - Pankaj Chandiramani, Shefali Bhargava, Jyoti Verma, Nithin Thekkupadam Narayanan
* **Last Updated By/Date** - Sambit Panda, Consulting Member of Technical Staff, Sep 2026
