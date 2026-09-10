# lambda-deploy

Deploy image and update lambdas

## `force_rebuild`

Por default la action no reconstruye si ya existe una imagen con el tag pedido
(normalmente el SHA del commit), y sólo re-apunta las lambdas a esa imagen.

Eso deja de servir cuando el contenido cambió por debajo de un SHA que no se movió:
el caso típico es una ronda de parches sobre la imagen base (`lambda-nodejs`). Ahí un
`workflow_dispatch` sale en verde sin haber reconstruido nada.

Para esos casos, pasar `force_rebuild: "true"`. La action entonces:

- saltea el chequeo del tag existente;
- **pushea con el tag `<sha>-rebuild-<run_id>.<attempt>`**, no con el SHA pelado. Los
  repos ECR de lambdas se crean `IMMUTABLE` (ver `modules/ecr-lambdas` en
  `meetlara/terraform`), así que re-pushear el mismo tag fallaría con
  `ImageTagAlreadyExistsException`;
- agrega `--pull`, para re-resolver la base mutable en vez de reusar el digest que
  quedó cacheado.

Las lambdas quedan apuntando al tag nuevo, que es también el que va a leer terraform
(el módulo resuelve la imagen por la última pusheada al repo).

Ojo: la lifecycle policy del repo conserva las últimas 3 imágenes, así que cada
rebuild forzado consume un slot.
