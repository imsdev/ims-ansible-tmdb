save-as-dataset
=========

Copies file from USS to z/OS dataset. By default, the file to copy is assumed to be at `uss_file_path`.

Requirements
------------

Must be run as system administrator

Role Variables
--------------

* uss_file_path
* MVS_uss_utilities_path
* file_name - the name of the file to copy from the `uss_file_path` path
* dataset_pds - all of the dataset HLQs for the save location
* dataset_member - the name to save the copied file `file_name` as

Dependencies
------------

No dependencies

Example Playbook
----------------

    - name: Save generate JCL as a dataset
      include_role:
        name: save-as-dataset
      vars:
        file_name: 'DFSLDCAT.jcl'
        dataset_pds: '{{ DFS_AUTH_LIB_HLQ1 }}.{{ DFS_AUTH_LIB_HLQ2 }}.INSTALL'
        dataset_member: 'IV3E319K'

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
