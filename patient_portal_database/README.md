# Patient Portal Database (PostgreSQL)

This container hosts the PostgreSQL database for the Patient 360 Portal. It stores users, patient demographics, encounters, vitals, medications, and chatbot transcripts.

Connection
- Use the connection command saved by the container: the file db_connection.txt contains the exact psql command.
- Current connection (from db_connection.txt):
  psql postgresql://appuser:dbuser123@localhost:5000/myapp

- Alternatively, via individual flags:
  psql -h localhost -p 5000 -U appuser -d myapp

Schema Overview
Tables:
1) users
   - id serial primary key
   - email varchar(255) unique not null
   - password_hash text not null
   - role varchar(50)
   - created_at timestamptz default now()

2) patients
   - id serial primary key
   - first_name varchar(100)
   - last_name varchar(100)
   - dob date
   - gender varchar(20)
   - contact jsonb
   - created_at timestamptz default now()

3) encounters
   - id serial primary key
   - patient_id int references patients(id) on delete cascade
   - type varchar(50)
   - date date
   - notes text

4) vitals
   - id serial primary key
   - patient_id int references patients(id) on delete cascade
   - recorded_at timestamptz
   - bp_systolic int
   - bp_diastolic int
   - heart_rate int
   - temperature numeric(4,1)

5) medications
   - id serial primary key
   - patient_id int references patients(id) on delete cascade
   - name varchar(120)
   - dose varchar(60)
   - frequency varchar(60)
   - start_date date
   - end_date date

6) chatbot_transcripts
   - id serial primary key
   - patient_id int references patients(id)
   - user_id int references users(id)
   - timestamp timestamptz default now()
   - message_role varchar(20) not null
   - message_text text not null

Indexes:
- patients(last_name)
- encounters(patient_id, date desc)
- vitals(patient_id, recorded_at desc)

Grants:
- appuser granted SELECT, INSERT, UPDATE, DELETE on all tables in schema public
- appuser granted USAGE, SELECT on all sequences in schema public

Seed Data
The database includes example data:
- 3 users: admin@example.com, clinician@example.com, user@example.com
- 3 patients with JSON contact details
- Sample encounters, vitals, and medications per patient
- 2 chatbot transcript messages associated to a patient and users

Maintenance Tips
- Backup: use the provided backup_db.sh to create a backup.
- Restore: use restore_db.sh to restore from a backup.
- DB Visualizer: source db_visualizer/postgres.env and run the Node viewer in db_visualizer if desired.

Notes
- All schema and seed operations were executed via one-at-a-time CLI psql -c statements.
- If you change ports, users, or passwords, ensure db_connection.txt and db_visualizer/postgres.env are updated by re-running startup.sh or updating values accordingly.
