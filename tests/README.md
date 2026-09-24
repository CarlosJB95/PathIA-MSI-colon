# tests/

Pruebas automáticas con `pytest` (ejecutar desde la raíz del repo: `pytest`).

Prioridad sugerida: pruebas de **integridad de los manifests**, que son las que
protegen la validez del estudio (ver sección *Convención de manifests* en el
README principal). Por ejemplo: que ningún `patient_id` aparezca en más de un
split, y que toda laminilla y todo parche pertenezcan a un paciente registrado.
