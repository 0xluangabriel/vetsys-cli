# VetSys — Veterinary Clinic Management System

[Português](README.md) | **English**

A command-line system in pure Python, with no external dependencies, for registering pet owners and animals, scheduling procedures, and keeping vaccination records and medical history. It models relational data without a database, using lists of dictionaries linked by ID and JSON persistence.

> The program's interface and code are in Portuguese, so menus, variable names and data keys below appear in Portuguese. Their meaning is explained where they first appear.

---

## Features

- **Pet owners and animals:** full CRUD, with automatically generated IDs and each animal linked to an owner.
- **Cascade delete:** removes an owner, their animals and the linked procedures, showing beforehand how many records will be affected.
- **Procedures:** scheduling and editing of five types (Vaccination, Spaying/Neutering, Lab test, Consultation and Surgery), with date validation.
- **Queries:** vaccination record and medical history in chronological order, plus an animal profile showing the owner and the total number of procedures.
- **Persistence:** automatic save to `clinica.json` after every change, and loading at startup.

---

## Example

The menu options are: [1] animal and owner registration, [2] procedure scheduling, [3] vaccination record, [4] history and animal profile, and [5] exit. Below, the user opens the vaccination record of Thor, owned by Ana.

```
==================================================
                      VetSys
==================================================
  [1] Cadastro de animais e tutores
  [2] Agendamento de procedimentos
  [3] Carteira de vacinação
  [4] Histórico e ficha do animal
  [5] Sair
==================================================
Digite a opção desejada: 3
--------------------------------------------------
ID: 1 | NOME: Thor | ESPÉCIE: Cachorro | TUTOR: Ana
ID: 2 | NOME: Mel | ESPÉCIE: Gato | TUTOR: João
--------------------------------------------------
ID do animal: 1
CARTEIRA DE THOR (tutor: Ana)
VACINA: V8 | DATA: 05/11/2026
VACINA: Antirrábica | DATA: 20/12/2026
```

---

## Technical decisions

### Relational modeling by ID
The data lives in a dictionary with three lists (`tutores` for owners, `animais` for animals, and `procedimentos` for procedures), linked by ID as foreign keys:

```python
{"id": 1, "id_tutor": 1, "nome": "Thor", "especie": "Cachorro", "peso": 30.0}
{"id": 1, "id_animal": 1, "tipo": "Vacinação", "nome": "V8", "data": "2026-11-05"}
```

Storing the ID instead of a copy of the object keeps a single source of truth: editing an owner's phone number updates it everywhere in the system.

### ID generation
The next ID is `max(id) + 1`, not `len(list) + 1`. After deletions, `len()` can return an ID that already exists and corrupt the links between records.

```python
def proximo_id(lista):
    return max((item["id"] for item in lista), default=0) + 1
```

### Reusable lookup
`buscar_id()` ("find by ID") only finds and returns the record (or `None`), without printing or deciding anything. That is why the same function serves listing, editing, deleting and reports. Because it returns a reference to the original dictionary, editing changes the list directly.

### Cascade delete without orphan records
Deletion goes through three levels: procedures, animals and, finally, the owner. To remove items, the code builds a new list with the items that remain, instead of removing them while iterating, which would skip consecutive elements.

```python
clinica["procedimentos"] = [p for p in clinica["procedimentos"] if p["id_animal"] not in ids_animais]
```

### ISO 8601 dates
Dates are entered as `dd/mm/yyyy`, validated with `datetime.strptime` (which rejects dates that do not exist, such as `31/02/2026`), and stored as `yyyy-mm-dd`. In this format, alphabetical order matches chronological order, so the history is sorted with a single `sort()`. Conversion back to the Brazilian format happens only on display.

### Centralized input validation
Each data type has its own input function, which repeats the question until it receives a valid value or returns `None`. This way, no invalid value, or a value left over from a previous iteration, gets through.

| Function | Guarantees |
|---|---|
| `ler_int()` | A valid integer |
| `ler_texto()` | A non-empty field |
| `ler_telefone()` | Digits only, keeping leading zeros |
| `ler_peso()` | A decimal between 0 and 500 kg, with a comma or a dot |
| `ler_data()` | A date that exists and is not in the past |

### Data-driven configuration
The procedure types live in a configuration dictionary instead of five duplicated blocks of code. Each entry holds the type's name and, when the type needs one, the prompt for its specific name (vaccine, test or surgery):

```python
TIPOS_PROCEDIMENTO = {
    1: ("Vacinação", "nome da vacina"),
    2: ("Castração", None),
    3: ("Exame", "nome do exame"),
    4: ("Consulta", None),
    5: ("Cirurgia", "nome da cirurgia"),
}
```

Adding a new type takes one line, and the validation rules stay centralized in a single flow.

### Fault tolerance
- **Safe loading:** the program catches `FileNotFoundError` and `json.JSONDecodeError`, so it starts even when the file is missing or corrupted.
- **Compatibility:** `setdefault()` recreates missing keys, keeping files from earlier versions compatible.
- **Broken links:** queries show `(tutor removido)` ("owner removed") instead of crashing.

---

## Concepts applied

Lists of dictionaries, functions with return values, nested loops, the filter pattern (`for` + `if`), `while True` with `break`, `try`/`except`, the `datetime` and `json` modules, context managers (`with open()`) and referential integrity.

---

## How to run

Requires only Python 3.

```bash
git clone https://github.com/0xluangabriel/vetsys-cli.git
cd vetsys-cli
python3 main.py
```

On Windows, use `py main.py`. The `clinica.json` file is created on the first registration.

---

Developed by **Luan Gabriel Alves da Silva** — [github.com/0xluangabriel](https://github.com/0xluangabriel)
