# dagster

#Goal Check the ping status of servers
#Following steps only applicable for chnaging default repo

cd /root/dagster_prod/dagster_prod

#Create backup of exiting repository.py

#Create new repository.py
#add the following content

mkdir /root/dagster_prod/dagster_prod/scripts
Create files morning.sh , host.txt in scripts add contents

from dagster import job, op, repository
import subprocess

@op
def run_morning_script():
    result = subprocess.run(
        #["/bin/bash", "/root/dagster_prod/scripts/morning.sh"],
        ["/bin/bash", "/root/dagster_prod/dagster_prod/scripts/morning.sh"],
        capture_output=True,
        text=True
    )
    if result.returncode != 0:
        raise Exception(f"Script failed: {result.stderr}")
    return result.stdout

@job
def ping_job():
    run_morning_script()

@repository
def my_repo():
    return [ping_job]



#reload repository.py from web UI , 

#check the details by launching the intence , it will give you the output folder path.

