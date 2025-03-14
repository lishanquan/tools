SEQUENCE



CREATE SEQUENCE IF NOT EXISTS wxwork_company_id_seq start with 100000000;

delete from pg_sequences WHERE sequencename = 'table_id_seq';
ERROR: cannot delete from view "pg_sequences"
详细：Views that do not select from a single table or view are not automatically updatable.
建议：To enable deleting from the view, provide an INSTEAD OF DELETE trigger or an unconditional ON DELETE DO INSTEAD rule.
TraceId : 0bc059cc17176557931692261e063a



https://www.modb.pro/db/58337



