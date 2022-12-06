Handling Git Submodules
=======================

The following is an overview of how to clone and update a repository that contains git submodules. The purpose of this document is to have a step by step guide on how to do it.

Cloning the Repository
----------------------

After cloning the repository normally, the submodules inside of it will be empty. To fetch all of the submodules we need to run the following 2 commands.

.. code-block:: console

    $ git submodule init
    $ git submodule update

This will populate the submodules with the last commit that was in the original modules (not the latest update from the submodules). 

Updating the Submodules
-----------------------

To update the submodules simply run the command:

.. code-block:: console
    
    $ git submodule update --remote

After running the three commands the submodules should be updated.
