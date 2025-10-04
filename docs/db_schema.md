<img width="1363" height="1172" alt="voting_diagram" src="https://github.com/user-attachments/assets/e4d264d1-78b3-4a04-b8d5-b8e751053e38" />


<summary>
  dbdiagram code
  <details>
    <pre><code>
 Table roles {
  id uuid [pk, default: `gen_random_uuid()`]
  code text [not null, unique] // ADMIN, USER
  name text [not null]
}

Table user_statuses {
  id uuid [pk, default: `gen_random_uuid()`]
  code text [not null, unique] // ACTIVE, BLOCKED, PENDING
  name text [not null]
}

Table users {
  id uuid [pk, default: `gen_random_uuid()`]
  email text [not null, unique] 
  password_hash text [not null]
  full_name text
  role_id uuid [not null, ref: > roles.id]
  status_id uuid [not null, ref: > user_statuses.id]
  created_at timestamptz [default: `now()`]
  updated_at timestamptz
}

Table voting_sessions {
  id uuid [pk, default: `gen_random_uuid()`]
  title text [not null]
  description text
  created_by uuid [ref: > users.id]
  start_at timestamptz [not null]
  end_at timestamptz [not null]
  is_published boolean [default: false]
  visibility text [default: 'public'] // public | restricted
  created_at timestamptz [default: `now()`]
  Note: 'Constraint: start_at < end_at'
}

Table voting_settings {
  session_id uuid [pk, ref: > voting_sessions.id] // 1:1 за счёт PK+FK
  anonymous boolean [default: true]
  multi_select boolean [default: false]
  max_choices int [default: 1]
  require_confirmed_email boolean [default: true]
  allow_vote_change_until_close boolean [default: false]
}

Table candidates {
  id uuid [pk, default: `gen_random_uuid()`]
  session_id uuid [not null, ref: > voting_sessions.id]
  full_name text [not null]
  program text
  order_no int [default: 0]

  indexes {
    (session_id)
    (session_id, full_name) [unique]
  }
}

Table votes {
  id uuid [pk, default: `gen_random_uuid()`]
  //session_id uuid [not null, ref: > voting_sessions.id]
  candidate_id uuid [not null, ref: > candidates.id]
  user_id uuid [not null, ref: > users.id]
  cast_at timestamptz [default: `now()`]
  weight numeric(10,2) [default: 1.00]
  is_valid boolean [default: true]

  indexes {
    //(session_id)
    (candidate_id)
    (user_id)
    (/*session_id,*/ user_id, candidate_id) [unique]
  }

  Note: 'Инварианты single/multi-select и совпадение session_id кандидат/голос — триггеры'
}

Table results {
  session_id uuid [pk, ref: > voting_sessions.id] // 1:1 за счёт PK+FK
  generated_at timestamptz [not null]
  total_votes int [not null]
  payload jsonb [not null] // агрегаты по кандидатам и метаданные
  signature text
}

Table notifications {
  id uuid [pk, default: `gen_random_uuid()`]
  user_id uuid [not null, ref: > users.id]
  type text [not null] // session_start | session_end | vote_accepted | system
  title text [not null]
  body text
  is_read boolean [default: false]
  created_at timestamptz [default: `now()`]

  indexes {
    (user_id)
    (is_read)
    (type)
  }
}

Table audit_logs {
  id uuid [pk, default: `gen_random_uuid()`]
  user_id uuid [ref: > users.id] // может быть NULL для неуспешных логинов
  action text [not null] // LOGIN | CREATE_SESSION | CAST_VOTE | ...
  entity_type text
  entity_id text
  meta jsonb
  created_at timestamptz [default: `now()`]

  indexes {
    (user_id)
    (created_at)
  }
}

</code>
</pre>
</details>
</summary>
