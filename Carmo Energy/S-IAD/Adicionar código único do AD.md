
Adicionar código único do AD para considerar a busca dos usuários para corrigir possível falha de ambiguidade dos usuários, considerar o campo de ID único do AD (**`objectGUID`**).

ALTERAÇÕES NO BANCO - BASE: dbSCE_06.05.2025

```sql
	ALTER TABLE usuario ADD usu_ds_guid_ad VARCHAR(256);
	ALTER TABLE usuario_auxiliar ADD usu_ds_guid_ad VARCHAR(256);
```

ALTERAÇÕES NA API:

Alterações nos métodos e adicionada coluna na model do usuário(usu_ds_guid_ad):

```c#
ValidateUser();
ValidateUserAd();
UpdateGeneral();
	UpdateUsuarioAuxiliar();
	UpdateUsuario();
```

