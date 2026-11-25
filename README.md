# caredesk

# CareDesk – Report Template (10–20 pages) & Oral Defense Cheat Sheet

## PART 1 – THESIS-STYLE REPORT BLUEPRINT

**Front matter (not counted):** title page (project title, your name, AUCA student ID, course, lecturer, date), declaration, abstract (120–150 words), table of contents, list of figures/tables.

### Chapter 1 – Introduction (1.5–2 pages)
- 1.1 Background: paper records in small hospitals cause lost files, double bookings, billing errors.
- 1.2 Problem statement: no single tool links patients, doctors, beds and bills with validation.
- 1.3 Objectives: general (build a GUI hospital manager in C/GTK 3) and specific (7–8 bullets: register, book, record, admit/discharge, bill, report, persist).
- 1.4 Scope and limitations: single user, local files, max 500 patients, one currency (RWF), no network.
- 1.5 Significance, 1.6 Report organisation (one line per chapter).

### Chapter 2 – Background / Literature (2 pages)
- 2.1 Hospital information systems (what they do; examples such as OpenMRS).
- 2.2 Why C: control, speed, teaches memory and structs. 2.3 Why GTK 3: free, cross-platform, event-driven; compare with WinAPI and FLTK in a small table.
- 2.4 Key concepts: event-driven programming, signals/callbacks, `GtkTreeView` model–view, binary file persistence, modular design.
- 2.5 Related work (2–3 existing systems) and what yours does simply.

### Chapter 3 – Design (3 pages)
- 3.1 Requirements: functional (the 10 features) and non-functional (readable, validated, saves automatically).
- 3.2 Architecture: draw the layers diagram `ui -> logic -> validation/storage -> models` (Figure 3.1).
- 3.3 Data design: table per struct (Patient, Doctor, Appointment, MedicalRecord, Admission, Bill, HistoryEntry) with fields and types; relationships (patient_id, doctor_id, appointment_id) as an ER diagram (Figure 3.2).
- 3.4 File format: `[int count][struct × count]` per `.dat` file.
- 3.5 Security design: login screen, `User` struct (username, password hash, role Admin/Staff), 3 attempts, default admin account on first run, passwords stored as a hash (djb2), only Admin can add users.
- 3.6 UI design: tab layout screenshot sketch, colour rules (red / orange / bold), dialog policy (every error and success is a pop-up).
- 3.7 Key algorithms in pseudocode: day-of-week (days since 1900-01-01, 1900-01-01 was Monday, `(n-1) % 7`), slot-conflict check, bill formula, occupancy %.

### Chapter 4 – Implementation (3–4 pages)
- 4.1 Environment: MSYS2, GCC, GTK 3.24, Makefile.
- 4.2 Module-by-module walk-through with 5–8 short code excerpts (≤15 lines each): `is_valid_date`, `schedule_appointment`, `generate_bill`, `refresh_patients_table` (row colours), `save_array/load_array`, a button handler.
- 4.3 Validation rules table (field → rule → message).
- 4.4 Challenges and solutions (e.g. dialogs asking confirmation before delete; discount recomputed from gross so it does not stack; bed freed by `is_active = 0`).
- 4.5 Packaging with DLLs (summary of BUILD_GUIDE.md, section D).

### Chapter 5 – Testing (2–3 pages)
- 5.1 Strategy: unit tests of `logic.c` (console, `tests/test_logic.c`) + manual GUI tests.
- 5.2 Test-case table (below) with columns: ID | Feature | Input | Expected | Actual | Pass/Fail. Fill "Actual" from your own run (screenshots of the dialogs in an appendix).
- 5.3 Results: "26 console checks passed" + GUI tests; list any defects fixed.

| ID | Feature | Input / action | Expected result |
|---|---|---|---|
| T1 | Register | Valid patient 1, DOB 1990-05-10, phone 0788123456 | Saved, confirmation dialog, row appears |
| T2 | Register | Same ID 1 again | Error "Patient ID 1 already exists." |
| T3 | Register | Empty name | Error "Full name cannot be empty." |
| T4 | Register | DOB 2090-01-01 | Error "cannot be in the future" |
| T5 | Register | Phone `078-812` | Error "digits only" |
| T6 | Register | DOB 1990-02-30 | Error "real date" |
| T7 | Doctor | ID 1, fee 10000, Mon+Wed | Added |
| T8 | Doctor | Fee 0 | Error "fee must be greater than 0" |
| T9 | Doctor | No day ticked | Error "Tick at least one day" |
| T10 | Appointment | Patient 1, Dr 1, 2030-01-07 (Mon), 08:00 | Confirmation with Appointment ID 1 |
| T11 | Appointment | Same doctor/date/slot again | Error "Slot already booked" |
| T12 | Appointment | 2030-01-06 (Sunday) | Error "Doctor not available on Sunday" |
| T13 | Appointment | Patient ID 99 | Error "does not exist" |
| T14 | Visit | Appointment still Scheduled | Error "only for Completed" |
| T15 | Visit | Follow-up 2030-01-01 (before visit) | Error "must be later than visit date" |
| T16 | Visit | Empty diagnosis | Error "Diagnosis cannot be empty" |
| T17 | Visit | Valid data after Completed | Saved; history table shows it |
| T18 | Admit | Patient 1, ICU, bed 3 | Status becomes Admitted (row red) |
| T19 | Admit | Another patient, ICU bed 3 | Error "bed already occupied" |
| T20 | Remove | Remove admitted patient | Error "currently admitted" |
| T21 | Discharge | Same day as admission | Days stayed = 1 (minimum), bed freed |
| T22 | Bill | Private insurance, fee 10000, ICU 1 day, medicine 5000, lab 2000 | Gross 97000, insurance 77600, net 19400 |
| T23 | Discount | 25 % | Error, bill unchanged |
| T24 | Discount | 10 %, Charity | Net becomes 17460, dialog shows it |
| T25 | Remove | Patient with unpaid bill | Error "unpaid bill" |
| T26 | Remove | After marking bill Paid, confirm Yes | Patient deleted, table refreshed |
| T27 | Persistence | Close and re-open program | All data still present |
| T28 | History filter | Patient ID = 1 | Only that patient's entries |
| T29 | Report | Generate + Export | Window shows figures; `report.txt` created |
| T30 | Login | Wrong password | Error "Wrong username or password", 2 attempts left |
| T31 | Login | 3 wrong attempts in a row | "Too many failed attempts", program closes |
| T32 | Login | admin / admin123 on first run | Main window opens, header shows "User: admin (Admin)" |
| T33 | Account | Change password with wrong current password | Error, password unchanged |
| T34 | Account | New password ≠ confirmation | Error "do not match" |
| T35 | Account | Valid change, then log out and log in with the old password | Old password rejected, new one accepted |
| T36 | Users | Admin adds staff user "nurse1" | Account created; duplicate name or password under 4 characters rejected |
| T37 | Users | Log in as Staff, try Add User | Error "Only an Admin can add users" |
| T38 | Logout | Click Logout, confirm | Login window appears again; history shows login/logout lines |
| T39 | Demo data | Click Yes on the first-run prompt | 5 doctors, 10 patients, 16 appointments, 4 admissions, 5 bills appear; admitted rows red, allergy rows orange, unpaid rows bold |
| T40 | Demo data | Load demo data a second time | Error "only into an empty system" |

### Chapter 6 – Conclusion (1 page)
Summary of what was achieved against each objective; limitations (single user, no login, bills aggregate all completed appointments, binary files not portable between compilers); future work (user accounts, SQLite database, printing, SMS reminders, charts).

**Back matter:** References (GTK docs https://docs.gtk.org/gtk3/, Kernighan & Ritchie, etc.), Appendix A screenshots, Appendix B selected code, Appendix C full test log.

---

## PART 2 – ORAL DEFENSE CHEAT SHEET

### 30-second pitch
"CareDesk is a GTK 3 desktop program in C. Data lives in fixed arrays of structs, saved to binary files. The rules are in `logic.c`, which knows nothing about GTK; `ui.c` only reads boxes, calls logic and shows dialogs."

### Demo data (new)
- `seed.c` does not write into the arrays directly: it calls `add_patient`, `schedule_appointment`, `generate_bill`... so demo data obeys every rule and appears in the Activity History.
- Dates come from `date_plus_days(offset, out)` (today + offset), so "today" and "this month" figures in the Report are never empty.
- Only an Admin can load it, and only when there are no doctors or patients (prevents ID clashes).

### Login (new)
- `main()` loops: `show_login_dialog()` → `create_main_window()` → `gtk_main()`. Logout closes the window and the loop shows the login again; closing with X ends the program.
- `login_user` hashes the typed password with `hash_password` (djb2: `hash = hash*33 + character`) and compares it to the stored hash. We never store the real password. The same error is shown for wrong user and wrong password.
- First run: `ensure_default_admin` creates `admin / admin123` if `users.dat` is empty. Change it in the Account tab.
- Limits to admit: djb2 is not a strong hash and has no salt; a real system would use bcrypt/argon2. Roles only control "Add User".

### Data flow (say this for any feature)
`button click` → `on_..._clicked` reads entries (`read_entry`, `read_int_field`) → calls a `logic.c` function → function validates, changes array, calls `log_activity`, returns 1/0 and a message → UI shows dialog → on success `after_change()` = `save_all_data()` + `refresh_all_tables()`.

### Key functions
| Function | What it does / why |
|---|---|
| `is_valid_date` | Checks length 10, `-` positions, digits, month 1–12, day ≤ days in month (leap years handled). |
| `date_to_day_number` | Counts days since 1900-01-01. Used by `day_of_week` ((n-1)%7, 0 = Monday) and `days_between`. |
| `is_future_date` | `strcmp(date, today) > 0` – works because YYYY-MM-DD sorts like real dates. |
| `calculate_age` | Year difference, minus 1 if birthday not yet reached. |
| `add_patient` | Validation order: ID unique → name → DOB valid → not future → phone digits → capacity; then copy struct, status "Outpatient". |
| `schedule_appointment` | Patient/doctor exist → valid date/slot → weekday check against `available_days` → loop for same doctor+date+slot (cancelled ignored) → `id = count + 1`. |
| `record_visit` | Appointment must be Completed, one record per appointment, diagnosis not empty, follow-up date text > visit date. |
| `admit_patient` / `discharge_patient` | Bed free = no active admission with same ward+bed. Discharge sets `is_active = 0` (frees bed), `days = max(1, days_between)`. |
| `generate_bill` | Consultation = sum of fees of Completed appointments; beds = days × ward rate; + medicine + lab = gross; insurance % (0/50/80); net = gross − covered. Reuses the patient's unpaid bill if one exists. Blocked while patient is still admitted. |
| `apply_discount` | 5–20 only; computed from (gross − insurance) so repeating a discount replaces, not stacks. |
| `can_remove_patient` | Blocks Admitted or unpaid; UI asks "Are you sure?" only if allowed. |
| `save_array` / `load_array` | Write `count` then the struct array with `fwrite`; on first run `fopen` fails → count = 0, no crash. |
| `refresh_patients_table` | Clears the `GtkListStore`, loops the array, picks colour (red admitted > orange allergies) and bold weight (unpaid). |
| `make_table` | Builds a `GtkTreeView`; each column uses a `GtkCellRendererText` bound to a store column. |

### Likely questions and short answers
1. **Why separate logic from UI?** Testable without a window (`tests/test_logic.c`), easier to explain, UI can change without breaking rules.
2. **Why arrays, not linked lists?** Simple, fixed limits, direct indexing; enough for the project (limit stated in scope).
3. **How do callbacks work?** `g_signal_connect(button, "clicked", G_CALLBACK(fn), NULL)` – GTK calls `fn` when the button is clicked; `gtk_main()` is the event loop.
4. **How does `GtkTreeView` get data?** Model–view: the `GtkListStore` holds the rows, the TreeView only displays; we clear and refill after every change.
5. **What if the .dat file is missing/corrupt?** `load_array` returns 0 and sets count to 0.
6. **How do you stop double booking?** Loop over appointments for the same doctor, date and slot with status not Cancelled.
7. **How is the weekday found?** 1900-01-01 was Monday; (days since − 1) mod 7.
8. **Why can't you delete a patient with an unpaid bill?** Money is still owed; `has_unpaid_bill` and `find_unpaid_bill` both checked.
9. **Limitations?** No login, binary files tied to the same struct layout (recompile = delete old `.dat`), bills sum all completed appointments (a second bill after payment would count them again).
10. **How would you improve it?** SQLite, user roles, per-appointment billed flag, printing.

### Be ready to modify live
- New ward: change `NUM_WARDS`, `ward_names`, `ward_rates` (logic.c / models.h).
- New insurance rate: `insurance_percent()` in logic.c.
- New field in Patient: add to struct, form entry in `build_patients_tab`, read in `on_add_patient_clicked`, column in `refresh_patients_table` (and delete old `.dat`).
