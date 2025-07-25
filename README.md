# goscraper

This project is a simple Golang API that fetches HTML content from a specified URL and parses it dynamically using regex. The API is designed for performance, speed, and accuracy, accommodating changes in the HTML structure.

> [!TIP]
> GoScraper is now open-sourced and self-deployable. Run your own goscraper instance
>
>  - `SUPABASE_URL` and `SUPABASE_KEY` should be the same creds as the front-end env variables used to access Supabase
>  - `ENCRYPTION_KEY` is an unique hash that is used to encrypt your data while saving the data to supabase.
> `openssl rand --hex 32` run this on shell to get a random hex key used for encryption
> - `VALIDATION_KEY` is used to validate the request from front-end, so use the same key as front-end has. a different key will reject requests from your front-end
>  - `URL` are the urls which the backend should allow to request. **(CORS)**

--- 

### WHEN DEPLOYING
Go to `globals/DevMode` and set the variable to false

---

### `.env`
```
SUPABASE_URL=""
SUPABASE_KEY=""
ENCRYPTION_KEY=""
VALIDATION_KEY=""
URL=""
```
---
## Supabase setup
### goscrape
```sql 
create table public.goscrape (
  "regNumber" text not null,
  "user" text null,
  timetable text null,
  courses text null,
  attendance text null,
  marks text null,
  "lastUpdated" numeric null,
  token text not null,
  ophour text null default ''::text,
  constraint goscrape_pkey primary key ("regNumber", token),
  constraint goscrape_regNumber_key unique ("regNumber")
);
```

### gocal
```sql 
create table public.gocal (
  id bigint generated by default as identity not null,
  date text null,
  month text null,
  day text null,
  "order" text null,
  event text null,
  created_at numeric null
);
```

### CRON Jobs
> [!WARNING]
> Install CRON Integration in supabase
> <img width="891" alt="image" src="https://github.com/user-attachments/assets/ad98da2f-f84c-43a9-8d87-8c512cdcaaa3" />


- Deleting old user data
```
  UPDATE goscrape
  SET "user" = NULL, timetable = NULL, attendance = NULL, marks = NULL, courses = NULL
  WHERE "lastUpdated" < (EXTRACT(EPOCH FROM NOW()) * 1000) - (12 * 60 * 60 * 1000);
```

- Deleting old calendar
```
CREATE OR REPLACE FUNCTION delete_from_gocal()
RETURNS void AS $$
BEGIN
    DELETE FROM gocal;
END;
$$ LANGUAGE plpgsql SECURITY INVOKER;

select
  cron.schedule (
    '0 0 * * *',
    'SELECT delete_old_calendar_events()'
  );
```
---

## Setup Instructions

1. **Clone the repository:**
   ```
   git clone https://github.com/rahuletto/goscraper
   cd goscraper
   ```

2. **Install dependencies:**
   ```
   go mod tidy
   ```

3. **Development Run the application:** (DEV SERVER)
   ```
   go run main.go
   ```

3. **Build and Run the application:** (BUILD SERVER)
   ```
   go build main.go
   ./main
   ```

## Docker file
```
docker compose up --build
```

## Contributing

Contributions are welcome! Please open an issue or submit a pull request for any enhancements or bug fixes.
