# Documentação do PostgreSQL

PROCEDURES / FUNCTIONS
```
SELECT
	CASE 
		WHEN p.prokind = 'f' THEN 'Function'
		WHEN p.prokind = 'p' THEN 'Procedure'
	END AS "Tipo",
	n.nspname AS "Schema",
	p.proname AS "Nome",
	pg_get_function_arguments(p.oid) AS "Argumentos",
	d.description AS "Comentário"
FROM pg_proc p
	LEFT JOIN pg_namespace n ON p.pronamespace = n.oid
	LEFT JOIN pg_language l ON p.prolang = l.oid
	LEFT JOIN pg_type t ON t.oid = p.prorettype 
	LEFT JOIN pg_description d ON d.objoid = p.oid
WHERE n.nspname not in ('pg_catalog', 'information_schema')
	AND p.prokind IN ('f','p')
ORDER BY 1, 2, 3;
```

TABLES / VIEWS
```		
SELECT
	CASE
		WHEN c.relkind = 'r' THEN 'Table'
		WHEN c.relkind = 'v' THEN 'View'
	END AS "Tipo",
	n.nspname AS "Schema",
	c.relname AS "Tabela",
	(SELECT obj_description(c.oid, 'pg_class')) AS comentario  
FROM pg_catalog.pg_class c 
	LEFT JOIN pg_catalog.pg_user u ON u.usesysid = c.relowner 
	LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace 
WHERE c.relkind IN ('r','v') 
	AND n.nspname NOT IN ('pg_catalog', 'pg_toast', 'information_schema')
ORDER BY 1, 2, 3;
```

FOREIGN KEYS
```
SELECT
	n.nspname AS "Schema",
	cl.relname AS "Tabela",
	a.attname AS "Coluna",
	ct.conname AS "Nome da chave",
	nf.nspname AS "Schema (referenciado)",
	clf.relname AS "Tabela (referenciada)",
	af.attname AS "Coluna (referenciada)"
FROM pg_catalog.pg_attribute a
	JOIN pg_catalog.pg_class cl ON (a.attrelid = cl.oid AND cl.relkind = 'r')
	JOIN pg_catalog.pg_namespace n ON (n.oid = cl.relnamespace)
	JOIN pg_catalog.pg_constraint ct ON (a.attrelid = ct.conrelid AND ct.confrelid != 0 AND ct.conkey[1] = a.attnum)
	JOIN pg_catalog.pg_class clf ON (ct.confrelid = clf.oid AND clf.relkind = 'r')
	JOIN pg_catalog.pg_namespace nf ON (nf.oid = clf.relnamespace)
	JOIN pg_catalog.pg_attribute af ON (af.attrelid = ct.confrelid AND af.attnum = ct.confkey[1])
ORDER BY 1, 2, 3;
```
