# archiver.py
import psycopg2
from datetime import datetime, timedelta

DB_PARAMS = {
    "dbname": "archiver_db",
    "user": "postgres",
    "password": "password",
    "host": "localhost",
    "port": 5432
}

def archive_old_logs(days=30):
    conn = psycopg2.connect(**DB_PARAMS)
    cur = conn.cursor()
    cutoff = datetime.now() - timedelta(days=days)

    # Move old logs to archive table
    cur.execute("""
        INSERT INTO logs_archive (id, message, created_at)
        SELECT id, message, created_at FROM logs WHERE created_at < %s;
    """, (cutoff,))
    cur.execute("DELETE FROM logs WHERE created_at < %s;", (cutoff,))
    conn.commit()
    cur.close()
    conn.close()
    print(f"Archived logs older than {days} days.")

if __name__ == "__main__":
    archive_old_logs(15)
