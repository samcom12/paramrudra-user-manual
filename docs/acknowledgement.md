# Accounts & Acknowledgement

## Creating an account

To get access to PARAM Rudra, register on the NSM user portal:

1. Register at **<https://services.nsmindia.in/userportal/account>** with your
   **official institutional email**, city and institute name.
2. Verify your email; a registration-form link is sent. Preview/edit, then
   submit.
3. Upload the required documents (ID proof, User Creation Form, etc.). *Once
   uploaded, documents cannot be modified — they go for verification.*
4. Your **HOD / PI / Co-PI** verifies your details and documents.
5. The coordinator selects the appropriate cluster; **higher authority grants
   final approval**.
6. You receive an email with your **user id, temporary password**, and allocated
   cluster.

!!! warning "Use your official email"
    Registrations from personal email addresses may be declined. Queries:
    `nsmsupport@cdac.in`. Help, FAQ and flowcharts are in the User Creation
    Portal's Help section.

Then follow [Getting Access](access.md) to log in and set your password (2FA via
Google Authenticator is mandatory).

## Accounting (CPU-hour allocation)

Usage is tracked like a banking system: each project/department is allocated a
budget of core-hours (typically per quarter, reset half-yearly). As jobs run, the
allocation is drawn down.

- Every job must charge a valid account: `#SBATCH -A <account>`.
- List your account(s):
  ```bash
  sacctmgr show assoc user=$USER format=account,partition,qos -p
  ```
- Inspect a job's resource usage:
  ```bash
  sacct -j <jobid> --format=JobID,JobName,State,Elapsed,AllocCPUS,MaxRSS,ExitCode
  ```

## QoS / fair-use policy (summary)

- Up to **10 simultaneous jobs** per user.
- Default max walltime **4 days**/job; **default walltime is 2 hours** if you
  don't specify `--time`.
- Indicative node × time trade-offs: 8-node job / 4 days, 16-node / 2 days,
  32-node / 1 day (subject to per-partition limits — see [Batch System](batch.md)).
- Need more than 4 days or more nodes? Raise a ticket; handled case-by-case.

See [Policies](policies.md) for the full rules.

## Acknowledging NSM in publications

If you use PARAM Rudra for any published work (theses, conference/journal papers,
patents), **please acknowledge the National Supercomputing Mission**. Include:

> We acknowledge National Supercomputing Mission (NSM) for providing computing
> resources of 'PARAM RUDRA' at C-DAC, No. 68, Electronic City Phase I,
> Konappana Agrahara, Bengaluru, Karnataka - 560100, which is implemented by
> C-DAC and supported by the Ministry of Electronics and Information Technology
> (MeitY) and Department of Science and Technology (DST), Government of India.

Please also send copies of dissertations, reports, reprints and URLs that
acknowledge *"National Supercomputing Mission, Government of India"* to:

```text
HPC Technologies,
Centre for Development of Advanced Computing,
CDAC Innovation Park, S.N. 34/B/1,
Panchavati, Pashan,
Pune – 411008, Maharashtra
```

Reporting your achievements helps NSM measure outcomes and justify future
augmentation of resources.

## Closing your account

When your research is complete and you no longer need PARAM Rudra, please close
your account:

1. Raise a ticket at **<https://paramrudra.cdac.in/support>**.
2. The system administrator will guide you through the closure procedure.
3. You will need clearance from your project coordinator / supervisor / HOD to
   obtain a "no dues" certificate from your institute.

Next: [FAQ & Support](support.md).
