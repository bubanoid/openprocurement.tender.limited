.. _tutorial:

Tutorial
========

Creating tender for reporting procedure
-----------------------------------------

Let's try creating tender with some data, passing the `procuringEntity` of a tender:

.. include:: tutorial/create-tender-procuringEntity.http
   :code:

We have `201 Created` response code, `Location` header and body with extra `id`, `tenderID`, and `dateModified` properties.

Let's check what tender registry contains:

.. include:: tutorial/tender-listing-after-procuringEntity.http
   :code:

We do see the internal `id` of a tender (that can be used to construct full URL by prepending `http://api-sandbox.openprocurement.org/api/0/tenders/`) and its `dateModified` datestamp.

Modifying tender
----------------

Let's update tender by supplementing it with all other essential properties:

.. include:: tutorial/patch-items-value-periods.http
   :code:

.. XXX body is empty for some reason (printf fails)

We see the added properies have merged with existing tender data. Additionally, the `dateModified` property was updated to reflect the last modification datestamp.

Checking the listing again reflects the new modification date:

.. include:: tutorial/tender-listing-after-patch.http
   :code:


.. index:: Document

Uploading documentation
-----------------------

Procuring entity can upload PDF files into the created tender. Uploading should
follow the :ref:`upload` rules.

.. include:: tutorial/upload-tender-notice.http
   :code:

`201 Created` response code and `Location` header confirm document creation. 
We can additionally query the `documents` collection API endpoint to confirm the
action:

.. include:: tutorial/tender-documents.http
   :code:

The single array element describes the uploaded document. We can upload more documents:

.. include:: tutorial/upload-award-criteria.http
   :code:

And again we can confirm that there are two documents uploaded.

.. include:: tutorial/tender-documents-2.http
   :code:

In case we made an error, we can reupload the document over the older version:

.. include:: tutorial/update-award-criteria.http
   :code:

And we can see that it is overriding the original version:

.. include:: tutorial/tender-documents-3.http
   :code:


.. index:: Enquiries, Question, Answer


Adding supplier information
---------------------------

Procuring entity can register supplier information:

.. include:: tutorial/tender-award.http
   :code:


Award confirmation
------------------

Procuring entity confirms awarding decision:

.. include:: tutorial/tender-award-approve.http
   :code:


Contract signing
----------------

Contract can be signed a day after award confirmation:


.. include:: tutorial/tender-contract-sign.http
   :code:


Cancelling tender
-----------------

Tender creator can cancel tender anytime. The following steps should be applied:

1. Prepare cancellation request
2. Fill it with the protocol describing the cancellation reasons 
3. Cancel the tender with the reasons prepared.

Only the request that has been activated (3rd step above) has power to
cancel tender.  I.e.  you have to not only prepare cancellation request but
to activate it as well.

See :ref:`cancellation` data structure for details.

Preparing the cancellation request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

  POST /tenders/{id}/cancellations


You should pass `reason`, `status` defaults to `pending`. `id` is
autogenerated and passed in the `Location` header of response.

.. code::

  Location: /tenders/{id}/cancellations/{cancellation-id}

Filling cancellation with protocol and supplementary documentation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Upload the file contents

.. code::

   POST /tenders/{id}/cancellations/{cancellation-id}/documents

Change the document description and other properties

.. code::

   PATCH /tenders/{id}/cancellations/{cancellation-id}/documents/{document-id}

Upload new version of the document

.. code::

   PUT /tenders/{id}/cancellations/{cancellation-id}/documents/{document-id}

Activating the request and cancelling tender
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

   PATCH /tenders/{id}/cancellations/{cancellation-id}
   
   {“data”:{“status”:”active”}}