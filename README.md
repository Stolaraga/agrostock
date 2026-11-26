# AgroStock — prototipo académico

Gestión inteligente de insumos agrícolas | UNED | Proyecto 03077.

## Inicio rápido

Abra `site/index.html` en un navegador e importe `site/agrostock_demo.csv` mediante **Importar CSV**. Para usar el botón de carga automática, ejecute desde la carpeta raíz `python -m http.server 8000 --directory site` y visite `http://localhost:8000`. No requiere instalar paquetes de JavaScript.

## Publicar en GitHub y Azure

1. Cree un repositorio GitHub llamado `agrostock`, preferiblemente privado, con rama `main`. Suba `site/`, `tests/` y `README.md` manteniendo la estructura (descomprima antes; no suba el ZIP como único archivo).
2. Ingrese a https://portal.azure.com/ y cree un recurso **Static Web App** (no Web App/App Service). Seleccione su suscripción, grupo `rg-agrostock-demo`, nombre `agrostock-demo` y plan **Free**. Verifique que no haya complementos de pago.
3. Origen GitHub; autorice la conexión y seleccione propietario, repositorio `agrostock` y rama `main`. Seleccione preset Custom. App location: `site`; API location: vacío; Output location: vacío. Cree el recurso.
4. En GitHub, abra `.github/workflows/azure-static-web-apps-*.yml` generado por Azure. En la acción `Azure/static-web-apps-deploy@v1`, confirme `app_location: "site"`, `api_location: ""`, `output_location: ""`, `skip_app_build: true`. Conserve el nombre del secreto generado por Azure y no publique el token. Si se requiere editar el YAML, confirme los cambios en main. Ejemplo de bloque en `azure-workflow-ejemplo.txt`.
5. En GitHub → Actions, compruebe que el despliegue terminó correctamente. En Azure → recurso → Overview, abra la URL pública asignada. Cargue el CSV y pruebe la entrada de 50 kg del fertilizante A, con fecha posterior a 2026-11-22 y no futura.

Documentación: https://learn.microsoft.com/en-us/azure/static-web-apps/get-started-portal y https://learn.microsoft.com/en-us/azure/static-web-apps/build-configuration?tabs=github-actions . El plan Free está sujeto a cuotas y a disponibilidad de la suscripción.

## CSV

UTF-8, encabezados obligatorios, fechas AAAA-MM-DD, cantidades con punto decimal (sin separador de miles). Se admite separador coma o punto y coma. Una fila `inicial` por insumo con cantidad inicial, plazo y reserva (enteros en semanas). Cada movimiento lleva ID único, código, nombre y unidad consistentes. Tipos: `entrada`, `salida`, `ajuste_entrada`, `ajuste_salida`, `cierre`. Los ajustes requieren una nota. Las filas `cierre` tienen cantidad vacía y fecha domingo: certifican una semana completa, incluso cuando no hay consumo. Las demás filas no iniciales dejan plazo y reserva vacíos. El sistema valida el CSV completo antes de reemplazar los datos. No se aceptan saldos negativos. No se permite editar silenciosamente semanas cerradas desde la interfaz; use ajustes posteriores documentados.

## Método predictivo

Se agregan las salidas por semana lunes-domingo y se utilizan solo las semanas cerradas consecutivas hasta la última semana calendario completa del corte global. El corte global es la última fecha registrada; no la fecha de apertura de la web. Con menos de 4 semanas no se estima; con 4–7 se usa el promedio de las 4 últimas; desde 8, se entrenan los dos métodos con todas las semanas salvo las 2 últimas, y se comparan mediante MAE en esas 2 semanas. Si la regresión mejora estrictamente el promedio y el próximo valor es positivo, se reentrena con todo el historial. De lo contrario, se usa el promedio reciente. Se truncan pronósticos negativos a cero. Se evita dar una fecha con consumo nulo, coeficiente de variación superior a 1 o si el agotamiento no aparece en 52 semanas. Se estima la cobertura desde el día posterior al corte, prorrateando la primera semana cuando es parcial. El umbral es la suma de consumos proyectados durante el plazo más la reserva. Estas reglas adicionales son supuestos de prototipo, no validados en campo.

## Caso de demostración

CSV ficticio con 94 filas y 5 insumos; corte 22/11/2026. Fertilizante A: 150 kg iniciales menos 120 kg de consumo = 30 kg, promedio 10 kg/semana, umbral 30 kg, cobertura 3 semanas, alerta. Fertilizante 15-15-15: 94 kg, regresión seleccionada, próxima semana 17 kg. Bioinsumo líquido: 6 semanas, promedio reciente; frijol: 2 semanas, sin predicción. Al agregar una entrada de 50 kg a A, se eleva el saldo a 80 kg y sale de la alerta de reposición (si el corte permanece en la misma semana).

## Pruebas y alcance

`node tests/engine.test.js` ejecuta pruebas del motor sin paquetes externos. Prototipo de demostración, sin autenticación, base de datos remota, sincronización entre equipos ni órdenes de compra automáticas. Los datos importados se guardan en localStorage del navegador; exporte periódicamente el CSV. No use datos confidenciales. El conjunto CSV incluido es público una vez desplegado. No se ha validado el pronóstico con una finca real, y el MAE de dos semanas es solo ilustrativo. El despliegue requiere acceso a su cuenta de Azure y GitHub.
