======================================
CozoScript 之外
======================================

Most functionalities of the Cozo database are accessible via the CozoScript API.
However, other functionalities either cannot conform to the "always return a relation" constraint,
or are of such a nature as to make a separate API desirable. These are described here.

The calling convention and even names of the APIs may differ on different target languages, please refer
to the respective language-specific documentation. Here we use the Python API as an example
to describe what they do.

.. module:: API
    :noindex:

.. function:: export_relations(self, relations)

    Export the specified ``relations``. It is guaranteed that the exported data form a consistent snapshot of 
    what was stored in the database.

    :param relations: names of the relations in a list.
    :return: a dict with string keys for the names of relations, and values containing all the rows.

.. function:: import_relations(self, data)
    
    Import data into a database. The data are imported inside a transaction, so that either all imports are successful, or none are.
    If conflicts arise because of concurrent modification to the database, via either CosoScript queries or other imports,
    the transaction will fail.

    The relations to import into must exist beforehand, and the data given must match the schema defined.

    This API can be used to batch-put or remove data from several stored relations atomically.
    The ``data`` parameter can contain relation names such as ``"rel_a"``, or relation names prefixed by a minus sign such as ``"-rel_a"``.
    For the former case, every row given for the relation will be ``put`` into the database, i.e. upsert semantics.
    For the latter case, the corresponding rows are removed from the database, and you should only specify the key part of the rows.
    As for ``rm`` in CozoScript, it is not an error to remove non-existent rows.

    .. WARNING::
        Triggers are not run for direct imports.

    :param data: should be given as a dict with string keys, in the same format as returned by ``export_relations``.
                 For example: ``{"rel_a": {"headers": ["x", "y"], "rows": [[1, 2], [3, 4]]}, "rel_b": {"headers": ["z"], "rows": []}}``


.. function:: backup(self, path)

    Backup a database to the specified path. 
    The exported data is guaranteed to form a consistent snapshot of what was stored in the database.

    This backs up everything: you cannot choose what to back up. It is also much more efficient than exporting all stored
    relations via ``export_relations``, and only a tiny fraction of the total data needs to reside in memory during
    backup.

    This function is only available if the ``storage-sqlite`` feature flag was on when compiling.
    The flag is on for all pre-built binaries except the WASM binaries.
    The backup produced by this API can then be used as an independent SQLite-based Cozo database.
    If you want to store the backup for future use, you should compress it to save a lot of disk space.

    :param path: the path to write the backup into. For a remote database, this is a path on the remote machine.

.. function:: restore(self, path)

    Restore the database from a backup. Must be called on an empty database. 
    
    This restores everything: you cannot choose what to restore.

    :param path: the path to the backup. You cannot restore remote databases this way: use the executable directly.

.. function:: import_from_backup(self, path, relations)
    
    Import stored relations from a backup.

    In terms of semantics, this is like ``import_relations``, except that data comes from the backup file directly,
    and you can only ``put``, not ``rm``. It is also more memory-efficient than ``import_relations``.

    .. WARNING::
        Triggers are not run for direct imports.

    :param path: path to the backup file. For remote databases, this is a path on the remote machine.
    :param relations: a list containing the names of the relations to import. The relations must exist
                        in the database.