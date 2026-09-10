# lambda-deploy

Deploy image and update lambdas

## `force_rebuild`

Por default la action no reconstruye si ya existe una imagen con el tag pedido
(normalmente el SHA del commit), y sólo re-apunta las lambdas a esa imagen.

Eso deja de servir cuando el contenido cambió por debajo de un SHA que no se movió:
el caso típico es una ronda de parches sobre la imagen base (`lambda-nodejs`). Ahí un
`workflow_dispatch` sale en verde sin haber reconstruido nada.

Para esos casos, pasar `force_rebuild: "true"`: saltea el chequeo del tag existente,
reconstruye pisando el tag y agrega `--pull` para re-resolver la base mutable en vez
de reusar el digest que quedó cacheado.
