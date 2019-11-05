submit-tso-rexx
=========

Used for the submission of rexx jobs that need to run in TSO/E address space.
This role runs the program `IKJEFT01`
By default, the REXX script to run should be located at {{ DFS_AUTH_LIB_HLQ1 }}.{{ DFS_AUTH_LIB_HLQ2 }}.{{ TSO_REXX_HLQ3 }}( {{ rexx_name }} )

Requirements
------------

Must be run as system administrator

Role Variables
--------------

* DFS_AUTH_LIB_HLQ1
* DFS_AUTH_LIB_HLQ2
* uss_file_path
* TSO_REXX_HLQ3 - third HLQ for REXX script save location
* command_result - read from check-job-status, command_result.stdout holds the submitted Job ID
* rexx_name - the name of the rexx script to call

Dependencies
------------

Depends on the following roles:

* submit-tso-rexx
* check-job-status
* save-as-dataset

Example Playbook
----------------

    - name: Submit REXX
      include_role:
        name: submit-tso-rexx
        public: yes
      vars:
        rexx_name: 'WAITJOB'

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).