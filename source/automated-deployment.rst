Automating Deployment with Puppet
=================================
Definitions
-----------
* BEAMLINE_WORKSTATION
    Any computer that beamline staff designate as a computer they want the
    collection stack installed on
* BEAMINE_SERVER
    The main "compute" computer where jupyterhub will point to
* IPYTHON_CONFIGURATION:
    The files that are used to configure the data collection environment.
    These have thus far been a collection of files that are prefixed with
    two numbers and are stored in ~/.ipython/profile_collection or a variant
* BACKUP_LOCATION
    Specific root folder for backing up IPYTHON_CONFIGURATION
* {{ BEAMLINE }}
    Placeholder syntax for any beamline ID, i.e., LIX, CHX, XPD, CSX, etc...
* {{ BEAMLINE }}_CONFIGURATION
    Conda package that includes the configuration information for at least
    metadatastore and filestore.  Optionally includes configuration information
    for a local channelarchiver server.  Other configuration information is
    welcome to be put in here.  BACKUP_LOCATION is something that should be
    added into the this configuration package
* COLLECTION
    Conda metapackage that enumerates the conda packages that make up our
    collection environment
* COLLECTION_{{ BEAMLINE }}
    Conda metapackages that are beamline specific and include any extra
    packages that a beamline needs or wants installed in their collection
    environment.
* ANALYSIS
    conda metapackage that enumerates the conda packages that make up our
    analysis environment
* ANALYSIS_{{ BEAMLINE }}
    conda metapackage that is beamline specific and includes at least the
    analysis metapackage and the {{ BEAMLINE }}_configuration conda package
* OLOG
    Logbook for everything {{ BEAMLINE }} related
* UAT
    User acceptance tests. These are the tests that, when successfully run,
    allows us to declare victory that our software stack is still working as
    intended on each beamline.  Note that these UAT's are the beamline's
    responsibility to create and maintain as it is just simply not possible
    for the DAMA group to know every way that each beamline is using the
    software stack.

Automated installation procedure for collection environment
-----------------------------------------------------------
1. Copy current state of IPYTHON_CONFIGURATION to BACKUP_LOCATION/year/(week-1)
   i.e., this is last week's IPYTHON_CONFIGURATION

2. Log that this backup occurred into the {{ BEAMLINE }} OLOG

3. Create new conda collection environment at BEAMLINE_WORKSTATION with the
   following command ::

       conda create -p /opt/conda_envs/collection-YEAR-WEEK COLLECTION_{{ BEAMLINE }} --override-channels -c anaconda -c nsls2-tag

4. Log that this conda collection environment was created.

5. Update the /opt/conda_envs/collection symlink to point to /opt/conda_envs/collection-YEAR-WEEK

6. **(Manual step)** Run UAT with the beamline staff.  If UAT fails, fix the
   problems on the spot or copy the stack trace to https://github.com/NSLS-II/Bug-Reports/issues
   and roll back the symlink so that it points at /opt/conda_envs/collection-YEAR-(WEEK-1)

Automated installation procedure for analysis environment(s)
------------------------------------------------------------
1. Create new conda analysis environment on BEAMLINE_WORKSTATION or
   BEAMLINE_SERVER with the following command ::

       conda create -p /opt/conda_envs/analysis-YEAR-WEEK ANALYSIS_{{ BEAMLINE }} --override-channels -c anaconda -c nsls2-tag

2. Log that this analysis environment was created into the {{ BEAMLINE }} OLOG

3. Diff the output of `conda list` for this new environment and the last one
   and log it to {{ BEAMLINE }} OLOG. If this diff is not empty, also send a
   message to the slack channel #auto-deploy ::

       diff <(conda list -p /opt/conda_envs/analysis-YEAR-(WEEK-1)) <(conda list -p /opt/conda_envs/analysis-YEAR-(WEEK-1))

4. Update the /opt/conda_envs/analysis symlink to point to /opt/conda_envs/analysis-YEAR-WEEK

5. Log that this symlink was updated
