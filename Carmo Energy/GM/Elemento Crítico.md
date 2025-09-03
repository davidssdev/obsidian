IMPLEMENTAÇÕES NO BANCO: dbSCE

Criação da tabela:

```sql
	CREATE TABLE gm_elemento_critico
	(
		elem_critico_cd_codigo INTEGER IDENTITY(1,1) NOT NULL PRIMARY KEY,
		ins_cd_instalacao INTEGER NOT NULL FOREIGN KEY REFERENCES instalacao (ins_cd_instalacao),
		elem_critico_cod_sap VARCHAR(2000) NOT NULL,
		elem_critico_descricao VARCHAR(2000) NOT NULL,
		elem_critico_dt_cadastro DATETIME NOT NULL,
		elem_critico_dt_atualizacao DATETIME,
		elem_critico_dt_inativo DATETIME,
		usu_cd_usuario INTEGER NOT NULL FOREIGN KEY REFERENCES usuario (usu_cd_usuario)
	);
```

Criação da consulta:

```SQL
CREATE PROCEDURE [dbo].[sp_gmListElementoCritico]
(
	@ins_cd_instalacao AS INTEGER = NULL,
	@ins_ds_instalacao AS VARCHAR(MAX) = NULL,
	@elem_critico_cd_codigo AS INTEGER = NULL,
	@elem_critico_descricao AS VARCHAR(MAX) = NULL
)
AS
BEGIN
		
		SELECT
			elemento.elem_critico_cd_codigo,
			elemento.elem_critico_cod_sap + ' - ' + elemento.elem_critico_descricao AS elem_critico_descricao,
			inst.ins_cd_instalacao,
			inst.ins_ds_instalacao
		FROM gm_elemento_critico elemento
			INNER JOIN instalacao inst ON (inst.ins_cd_instalacao = elemento.ins_cd_instalacao)
		WHERE 1 = 1
			AND (@elem_critico_cd_codigo IS NULL OR elemento.elem_critico_cd_codigo = @elem_critico_cd_codigo)
			AND (@ins_cd_instalacao IS NULL OR elemento.ins_cd_instalacao = @ins_cd_instalacao)
			AND (@elem_critico_descricao IS NULL OR elemento.elem_critico_descricao LIKE '%' + @elem_critico_descricao + '%')
			AND (@ins_ds_instalacao IS NULL OR inst.ins_ds_instalacao LIKE '%' + @ins_ds_instalacao + '%')
END;
```

Consulta Para o Excel:
	
```SQL
CREATE PROCEDURE [dbo].[sp_gmListElementoCriticoExcel]
AS
BEGIN
		
		SELECT
		instalacao.ins_ds_instalacao AS instalacao_ds,
		elemento.elem_critico_cod_sap AS elemento_cd_sap,
		elemento.elem_critico_descricao AS elemento_ds,
		'' AS elemento_excluir
		FROM instalacao
		INNER JOIN gm_elemento_critico elemento ON (elemento.ins_cd_instalacao = instalacao.ins_cd_instalacao)
		WHERE elemento.elem_critico_dt_inativo IS NULL
		ORDER BY instalacao.ins_ds_instalacao;

END
```

Alterações na tabela temporalidade:

```SQL
ALTER TABLE gm_temporalidade_info ADD gm_dt_temp_inativo DATETIME;

INSERT INTO gm_temporalidade_info (gm_ds_temp_info) SELECT 'Outros';

UPDATE gm_temporalidade_info SET gm_dt_temp_inativo = GETDATE() WHERE gm_cd_temp_info IN (2,3)

```

Alterações na consulta temporalidade:

```SQL
ALTER PROCEDURE [dbo].[sp_gmListTempInfo]
(
	@id AS INTEGER = NULL
)
AS
BEGIN
	SELECT
		tempinfo.gm_cd_temp_info, tempinfo.gm_ds_temp_info
	FROM gm_temporalidade_info tempinfo
	WHERE 1 = 1 
		AND (tempinfo.gm_dt_temp_inativo IS NULL OR tempinfo.gm_cd_temp_info = @id)
	ORDER BY tempinfo.gm_ds_temp_info
END
```

Alterações na tabela GM:

```SQL
ALTER TABLE gm ADD elem_critico_cd_codigo INTEGER FOREIGN KEY REFERENCES gm_elemento_critico (elem_critico_cd_codigo)
ALTER TABLE gm ADD elem_critico_cod_sap VARCHAR(2000)
ALTER TABLE gm ADD gm_ds_elemento_critico VARCHAR(2000)
```

Alterações na consulta da GM (sp_gmListGM):

```SQL
elem_critico_cd_codigo
gm.elem_critico_cod_sap,
gm.gm_ds_elemento_critico
```

Alterações consulta da GM (sp_gmListGMreport):

```SQL
CASE WHEN gm.gm_dt_fam_aprov IS NOT NULL AND gm.gm_cd_temp = 1 THEN DATEADD(DAY, gm.gm_qtd_duracao_dias, gm.gm_dt_fam_aprov) ELSE NULL END AS gm_dt_fim_prevista,
elemento.elem_critico_cd_codigo,
CASE WHEN gm.elem_critico_cd_codigo IS NOT NULL THEN elemento.elem_critico_cod_sap + ' - ' + elemento.elem_critico_descricao ELSE gm.elem_critico_cod_sap + ' - ' + gm.gm_ds_elemento_critico END AS elem_critico_descricao
```

Alterações consulta da GM (sp_gmGetExcelPBsepro):	

```SQL
[Envolve Elemento Crítico] AS envolveElementoCritico,
[Elemento Crítico] AS elementoCritico,
```

Adicionado campo na view do SEPRO (vw_pb_sepro):

```SQL
CASE WHEN (gm.elem_critico_cd_codigo IS NOT NULL OR gm.elem_critico_cod_sap IS NOT NULL) THEN 'SIM' ELSE 'NÃO' END AS [Envolve Elemento Crítico],
CASE WHEN gm.elem_critico_cd_codigo IS NOT NULL THEN elemento.elem_critico_cod_sap + ' - ' + elemento.elem_critico_descricao ELSE gm.elem_critico_cod_sap + ' - ' + gm.gm_ds_elemento_critico END AS [Elemento Crítico],
CASE WHEN gm.gm_dt_fam_aprov IS NOT NULL AND gm.gm_cd_temp = 1 THEN DATEADD(DAY, gm.gm_qtd_duracao_dias, gm.gm_dt_fam_aprov) ELSE NULL END AS gm_dt_fim_prevista
```