:author: Horea-Ioan Ioanas
:email: ioanas@biomed.ee.ethz.ch
:institution: Institute for Biomedical Engineering, ETH and University of Zurich
:corresponding:

:author: Bechara John Saab
:email: bechara.saab@uzh.ch
:institution: Preclinical Laboratory for Translational Research into Affective Disorders, DPPP, Psychiatric Hospital, University of Zurich

:author: Markus Rudin
:email: rudin@biomed.ee.ethz.ch
:institution: Institute for Biomedical Engineering, ETH and University of Zurich

:bibliography: mybib


LabbookDB - A Wet-Work-Tracking Database Application Framework
==============================================================

.. class:: abstract

    LabbookDB is a database application framework containing a relational database structure for life science research -
    along with a number of functions to conveniently add and retrieve information, and generate summaries.
    The core concept of LabbookDB is that most of the information commonly tracked in a lab book can be more efficiently and more reliably stored in a relational database.
    We overcome the lack of flexibility and portability of designed-for-analysis spreadsheets and databases by building the database schema around atomized physical interactions of objects in the laboratory
    (and providing ready-for-plotting and/or ready-for-analysis dataframes as a compatibility layer).
    LabbookDB provides a wet work metadata storage model excellently suited for explorative ex-post reporting and analysis, as well as a potential infrastructure for automated wet work tracking.

.. class:: keywords

    laboratory notebook, labbook, wet work, record keeping, internet of things, reports, life science, biology, neuroscience, behaviour, relational database, normalization, SQL

Introduction
------------

The laboratory notebook (more commonly, lab book) is a long-standing multi-purpose record -
serving as a primary data trace, as a calendar, diary, legal document, memory aid, organizer, timetable, and also proposed as a rapid science communication medium :cite:`Bradley2007`.
It is of notable popularity in the natural sciences, especially in the life sciences |---| where research largely consists of “wet work” (i.e. real-world manipulation), which generally leaves no data trace unless explicitly recorded.
With the advent of electronic data acquisition and storage, however, the lab book has increasingly lost significance as a repository for actual data, and has transformed into a metadata record.
Notably, the modern lab book has become a general repository of information, for which simple array formats (e.g. tables, spreadsheets, or data matrices) do not provide an adequate input and/or storage format.

Some scientists and science service providers seek to emulate the seemingly convenient functionality of the lab book in the electronic medium -
even providing support for sketching and doodling (e.g. eLabFTW :cite:`elabftw`).
Storing information in free-text or pictorial form, however, exacerbates the lab book's incompatibility with electronic data analysis and reporting
(which commonly requires consistent array formats).
This approach, rather than merely retarding information flow by increasing the need for manual lookup and input, can also increase the incidence of biased evaluation.
This can happen most easily as a consequence of notes being more often or more attentively consulted, and judged by varied but not explicitly documented standards, depending on the expectations of the researcher.

Conversely, some researchers routinely force multidimensional and relationship-rich experimental metadata into the familiar and analysis-friendly spreadsheet format.
Under a list-like model, however, relationships become spread over many combinations of cells while remaining untracked.
This leads to the repetition of identical cell combinations across multiple entries, which in turn renders e.g. the task of updating the correspondence between related cells non-trivial.
These issues are known as information redundancy and update anomalies, respectively -
and are prone to damage data integrity over time.
The temptation also arises in such cases to truncate the data input to only the parts considered essential at the time of the experiment.
This runs the risk of omitting information which may have been easily recorded (even automatically) given a proper data structure, and which may become crucial for closer ex-post deliberation of results.

The crux of the issue, which neither of these approaches adequately addresses, is to store experimental metadata in a fashion which befits its relationship-rich nature, while providing array-formatted data output for analysis, and spreadsheet-formatted data for human inspection.
Solutions which provide such functionality for a comprehensive experimental environment are few, and commonly proprietary and enterprise oriented (e.g. iRATS, REDCap :cite:`Harris2009`).
One notable exception is MouseDB :cite:`mousedb`, a database application framework built around mouse tracking.
This package is considerably more mature than our present endeavour, yet more closely intended as a lab management tool rather than a general lab book replacement.
It makes a number of differing structure choices, but given the permissive license (BSD :cite:`bsd`) of both projects, it is conceivable for functionalities from one to be merged into another in the future.

The need for a wet work metadata system which provides a better internal data model and consistently structured outputs, is compounded by the fact that such a system may also be better suited for (semi)automatic record keeping.
Some automated tracking (in a rudimentary barcode-scanning fashion) is already available for at least one commercial platform (iRATS), and the concept is likely to become of even more interest, as the internet of things reaches the laboratory.
This makes a well-formed open source relational schema of object interactions accurately representing the physical world pivotal in the development of laboratory record keeping.

Methods
-------

Database Management System
~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to cast complex laboratory metadata into tractable relationships with high enough entry numbers for statistical analysis, as well as to reduce data redundancy and the risk of introducing anomalies, we opt for a relational database management system, as interfaced with via SQLAlchemy.
The scalability and flexibility advantages of flat data stores are not pertinent to the content at hand, as experimental metadata is comparatively smaller, more reliable, and more slowly obtained than data from other use cases which these systems were built to adress.
With an easily extendable but robust schema design we set out to reposit data in such a way that encourages standardization in reporting and comparability across experiments.

Database Schema Design
~~~~~~~~~~~~~~~~~~~~~~

The current database schema was generated from numerous bona fide spreadsheet formats used at the Psychiatry University Clinic, ETH, and University of Zurich.
Iteratively, these spreadsheets are being normalized to first, second, third, and fourth normal forms (eliminating multivalued attributes, partial dependencies, transitive dependencies, and multivalued dependencies, respectively) :cite:`Codd1974`.
As the database schema of the current release (``0.0.1``) consists of over 40 tables, and is expected to expand as more facets of wet work are tracked, ensuring that relationships are well-formed will remain an ongoing process.
The perpetually non-definitive nature of the database schema is also conditioned by the continuous emergence of new wet work methods.

Record Keeping and Structure Migration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We use version tracking via Git to provide both a verifiable primary input record, and the possibility to correct entries (e.g. typos) in order to facilitate later database usage in analysis.
Version tracking of databases, however, is rendered difficult by their binary format.
To mitigate this issue, as well as the aforementioned continuous structure update requirement, we track modular Python function calls which use the LabbookDB input application programming interface (API) to generate a database |---| instead of the database itself.
We refer to this repository of Python function calls as the “source code” of the database.

Input Design
~~~~~~~~~~~~

The LabbookDB input API consists of Python functions which interface with SQLAlchemy, and accept dictionary and string parameters for new entry specification and existing entry identification, respectively.
These Python functions are wrapped for command line availability via ``argh`` |---| as sub-commands under the master command ``LDB`` in order to conserve executable namespace.
Dictionaries are passed to the command line surrounded by simple quotes, and a LabbookDB-specific syntax was developed to make entry identification considerably shorter than standard SQL (though only arguably more readable).

Output Design
~~~~~~~~~~~~~

Outputs include simple human-readable command line reports and spreadsheets, ``.pdf`` protocols, introspective graphs, and dataframes.
Dataframe ouput is designed to support both the Pandas ``DataFrame`` format and export as ``.csv``.
The dataframe conventions are kept simple and are perfectly understood by BehavioPy :cite:`behaviopy`, a collection of plotting functions originally developed as part of LabbookDB, but now branched off for more general usage.
The formatting of command line reports is built by concatenating ``__str__`` methods of queryable objects and their immediate relationships, and is based on the most common use cases for rapid monitoring.
Contingent on the availability of object-specific formatting guidelines, an interface is available for generating human-readable, itemized ``.pdf`` protocols.

Scope
~~~~~

To accommodate for a developing schema, reduce dependencies, and reduce usage difficulty, we opt to showcase LabbookDB as a personal database system, using SQLite as an engine.
As such, the database is stored locally, managed without a client-server model, and accessed without the need for authentication.
The scope thus extends to maximally a few users, which trust each other with full access.
This is an appropriate scope for most research groups.
Aditionally, this design choice allows single researchers or clusters of computationally inclined researchers within a larger group to autonomously try out, test, contribute to, or adopt LabbookDB without the need for a larger institutional commitment.

Development Model
~~~~~~~~~~~~~~~~~

The database schema draws from ongoing input, testing, and the wet work experience of many researchers associated with the Institute of Biomedical Engineering and the Animal Imaging Center at the ETH and University of Zurich.
The development team consists of one programmer (corresponding author), who will maintain and actively develop LabbookDB at least until 2019 |---| independently of community involvement.
Beyond that time point development may become contingent on the established impact of the project, including number of contributors, academic recognition of the metadata management system, adoption in the scientific Python or biomedical community, or the prospect of developing commercial tools to leverage the open source schema and API.

Capabilities
------------

To demonstrate the capabilities of this first LabbookDB release in a concise fashion, we provide example database source code in a separate repository, Demolog :cite:`demolog`.
Contingent on the presence of LabbookDB ``0.0.1`` and its dependencies on the system, the example database can be built |---| and correspondingly described subsequent entries can be executed locally.
To set up the example database, the following should be run from the terminal:

.. code-block:: console

    mkdir ~/src
    cd ~/src
    git clone https://bitbucket.org/TheChymera/demolog
    cd demolog/from_python_code
    ./generate_db.py
    mkdir ~/syncdata
    cp meta.db ~/syncdata

Note that, for the examples to work, it is mandatory to create the ``src`` and ``syncdata`` directories under the user's home path.

Entry Insertion and Update
~~~~~~~~~~~~~~~~~~~~~~~~~~

The Python API allows for clearly laid out entry insertion, via the ``add_generic()`` function:

.. code-block:: python

    add_generic(db_location, parameters={
        "CATEGORY":"Animal",
        "sex":"m",
        "ear_punches":"L",
        "license":"666/2013",
        "birth_date":"2016,7,21",
        "external_ids":[
            {"CATEGORY":"AnimalExternalIdentifier",
                "database":"ETH/AIC",
                "identifier":"5682",
                },
            {"CATEGORY":"AnimalExternalIdentifier",
                "database":"UZH/iRATS",
                "identifier":"M2889"
                },
            ],
        "genotypes":["Genotype:code.datg"],
        })

Technically, all entries could be created in such a fashion.
However, in order to better organize logging (e.g. quarterly, as in the Demolog submodules), we provide an additional function for entry update.
Instead of editing the original animal input file to set e.g. the death date, the animal entry can be updated via a separately tracked Python file:

.. code-block:: python

   append_parameter(db_location,
       entry_identification="Animal:external_ids."
           "AnimalExternalIdentifier:database."
           "ETH/AIC&#&identifier.5682",
       parameters={
           "death_date":"2017,5,13,17,25",
           "death_reason":"end of experiment",
           }
       )

In this example an existing entry is selected in a compact fashion using custom LabbookDB syntax.

Compact Syntax for Entry Selection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Custom LabbookDB syntax is not written as a wrapper for SQL, but rather specifically designed to satisfy LabbookDB entry selection use cases in a minimum number of characters.
This is primarily provided to facilitate database manipulation from the command line, though it also aids in making database source code more clearly laid out

Consider the string used to identify the entry to be updated in the previous code snippet (split to fit document formatting):

.. code-block:: python

   "Animal:external_ids.AnimalExternalIdentifier:datab"
   "ase.ETH/AIC&#&identifier.5682"

Under the custom LabbookDB syntax, the selection string always starts with the entry's object name (in the string at hand, ``Animal``).
The object name is separated from the name of the attribute to be matched by a colon, and the attribute name is separated from the value identifying the existing entry by a period.
The value can be either a string, or |---| if the string contains a colon |---| it is presumed to be another object (which is then selected by using the same syntax).
Multiple matching constraints can be specified, by separating them via double ampersands.
Inserting one or multiple hashtags in between the ampersands indicates at what level the additional constraint is to be applied.
In the current example, two ampersands separated by one hashtag mean that an ``AnimalExternalIdentifier`` object is matched contingent on a ``database`` attribute value of ``"ETH/AIC"`` and an ``identifier`` attribute value of ``"5682"``.
Had the ampersands not been separated by a hashtag, the expression would have prompted LabbookDB to apply the additional ``identifier`` attribute constraint not to the ``AnimalExternalIdentifier`` object, but one level higher, to the ``Animal`` object.

Command Line Reporting
~~~~~~~~~~~~~~~~~~~~~~

Quick reports can be generated directly via the command line, e.g. in order to get the most relevant aspects of an animal at a glance.
The following code should be executable locally in the terminal, contingent on LabbookDB example database availability:

.. code-block:: console

    LDB animal-info -p ~/syncdata/meta.db 5682 ETH/AIC

The code should return an overview similar to the flollowing, directly in the terminal:

.. code-block:: console

    Animal(id: 15, sex: m, ear_punches: L):
       license:	666/2013
       birth:	2016-07-21
       death:	2017-05-13 (end of experiment)
       external_ids:	5682(ETH/AIC), M2889(UZH/iRATS)
       genotypes:	DAT-cre(tg)
       cage_stays:
          cage 31, starting 2016-12-06
          cage 37, starting 2017-01-10
       operations:
          Operation(2017-03-04 10:30:00: virus_injection)
          Operation(2017-03-20 13:00:00: optic_implant)
       treatments:
       measurements:
          Weight(2016-12-22 13:35:00, weight: 29.6g)
          Weight(2017-03-30 11:48:00, weight: 30.2g)
          fMRI(2016-12-22 13:35:49, temp: 35.0)
          fMRI(2017-03-30 11:48:52, temp: 35.7)
          Weight(2017-04-11 12:33:00, weight: 29.2g)
          fMRI(2017-04-11 12:03:58, temp: 34.8)
          Weight(2017-05-13 16:53:00, weight: 29.2g)

Human Readable Spreadsheets
~~~~~~~~~~~~~~~~~~~~~~~~~~~

For a broader overview, LabbookDB can join tables from the database in order to construct comprehensive human-readable spreadsheet overviews.
Storing information in a well-formed relational structure allows for versatile and complex reporting formats.
In the following model, for instance, the “responsive functional measurements” column is computed automatically from the number of fMRI measurements and the number of occurrences of the ``"ICA failed to indicate response to stimulus"`` irregularity on these measurements.

Contingent on the presence of LabbookDB and the example database, the following lines of code should generate a dataframe formatted in the same fashion as Table :ref:`reporttab`, and return it directly in the terminal, or save it in ``.html`` format, respectively:

.. code:: console

    LDB animals-info ~/syncdata/meta.db
    LDB animals-info ~/syncdata/meta.db -s overview

An example of the ``.html`` output can be seen in the Demolog repository under the ``outputs`` directory.

.. raw:: latex

   \setlength{\tablewidth}{1.7\linewidth}

.. table:: Example of a human-readable overview spreadsheet generated via the LabbookDB command line functionality. :label:`reporttab`
   :class: w

   +-----------+---------+-----------+---------------+-------------------------+------------------------------------+
   | Animal_id | ETH/AIC | UZH/iRATS | Genotype_code |    Animal_death_date    | responsive functional measurements |
   +===========+=========+===========+===============+=========================+====================================+
   |    45     |   6258  |   M5458   |     datg      |   2017-04-20 18:30:00   |                0/0                 |
   +-----------+---------+-----------+---------------+-------------------------+------------------------------------+
   |    44     |   6262  |   M4836   |     eptg      |                  None   |                2/2                 |
   +-----------+---------+-----------+---------------+-------------------------+------------------------------------+
   |    43     |   6261  |   M4835   |     eptg      |   2017-04-09 18:35:00   |                0/0                 |
   +-----------+---------+-----------+---------------+-------------------------+------------------------------------+
   |    42     |   6256  |   M4729   |     epwt      |                  None   |                0/0                 |
   +-----------+---------+-----------+---------------+-------------------------+------------------------------------+
   |    41     |   6255  |   M4728   |     eptg      |                  None   |                2/2                 |
   +-----------+---------+-----------+---------------+-------------------------+------------------------------------+

Printable Protocol Output
~~~~~~~~~~~~~~~~~~~~~~~~~

LabbookDB can create ``.pdf`` outputs to serve as portable step-by-step instructions suitable for computer-independent usage.
This capability, paired with the database storage of e.g. protocol parameters means that one can store and assign very many protocol variants internally (with a minuscule storage footprint),
and conveniently print out a preferred protocol for collaborators, technicians, or students, without encumbering their workflow with any unneeded complexity.
The feature can be accessed from the ``labbookdb.report.examples`` module.
The following code should be executable locally, contingent on LabbookDB and example database availability:

.. code-block:: python

    from labbookdb.report.examples import protocol

    class_name = "DNAExtractionProtocol"
    code = "EPDqEP"
    protocol("~/syncdata/meta.db", class_name, code)

This should create a ``DNAExtractionProtocol_EPDqEP.pdf`` file identical to `the one tracked in Demolog <https://bitbucket.org/TheChymera/demolog/raw/9ce8ca3b808259a1cfe74169d7a91fb40e4cfd90/outputs/DNAExtractionProtocol_EPDqEP.pdf>`_.

Introspection
~~~~~~~~~~~~~

LabbookDB ships with a module which generates graphical representations of the complex relational structures implemented in the package.
The feature is provided by the ``labbookdb.introspection.schema`` module.
The following code should be executable locally, contingent on LabbookDB availability:

.. code-block:: python

    from labbookdb.introspection.schema import generate

    extent=[
        "Animal",
        "FMRIMeasurement",
        "OpenFieldTestMeasurement",
        "WeightMeasurement",
        ]
    save_plot = "~/measurements_schema.pdf"
    generate(extent, save_plot=save_plot)

This example should generate Figure :ref:`measurements` in ``.pdf`` format (though ``.png`` is also supported).

.. figure:: measurements_schema.pdf
    :scale: 52%

    LabbookDB schema section, illustrating the polymorphic relationship between Animal objects and different Measurement variants. :label:`measurements`

Polymorphic Mapping
~~~~~~~~~~~~~~~~~~~

In current research, it is common to subject animals to experimental procedures which are similar in kind, but which can be split into categories with vastly different attributes.
Prime examples of such procedures are ``Measurements`` and ``Operations``.
In Figure :ref:`measurements` we present how LabbookDB uses SQLAlchemy's support for polymorphic relationships to link different measurement types to the ``measurements`` attribute of the ``Animal`` object.
Attributes common to all measurement types are stored on the ``Measurement`` table, as are relationships common to multiple measurements (e.g. the relationship to the ``Animal`` object, instantiated in the ``animal_id`` attribute).

Atomized Relationships
~~~~~~~~~~~~~~~~~~~~~~

We use the expression “atomized relationships” to refer to the finest grained representation of a relationship which can feasibly be observed in the real world.
In more common relational model terms, higher atomization would correspond to higher normal forms |---| though we prefer this separate nomenclature to emphasize the preferential consideration of physical interactions, with an outlook to more automated wet work tracking.
Similarly to higher normal forms, increasingly atomized relationships give rise to an increasingly complex relational structure of objects with decreasing numbers of attributes.
LabbookDB embraces the complexity thus generated and the flexibility and exploratory power it facilitates.
Database interaction in LabbookDB is by design programmatic, an thus ease of human readability of the raw relational structure is only of subordinate concern to reporting flexibility.

An example of relationship atomization is showcased in Figure :ref:`cagestays`.
Here the commonplace one-to-many association between ``Cage`` and ``Animal`` objects is replaced by a ``CageStay`` junction table highlighting the fact that the relationship between ``Cage`` and ``Animal`` is bounded by time, and that while it is many-to-one at any one time point, in the overarching record it is, in fact, many-to-many.
This structure allows animals to share a cage for a given time frame, and to be moved across cages independently |---| reflecting the physical reality in animal housing facilities.
This complexity is seamlessly handled by LabbookDB reporting functions, as seen e.g. in the command line reporting example previously presented.

Conversely, atomization can result in a somewhat simpler schema, as higher level phenomena may turn out to be special cases of atomized interactions.
By design (and in contrast to `the MouseDB implementation <https://github.com/davebridges/mousedb/blob/49b0a2c4eb7008fb8ed663d6a05a96d52d2a6d6d/mousedb/animal/models.py#L276>`_), we would not track breeding cages as a separate entity, as the housing relationships are not distinct from those tracked by the ``CageStay`` Object.
A separate object may rather be introduced for breeding events |---| which need not overlap perfectly with breeding cages.


.. figure:: cagestay_schema.pdf

    LabbookDB schema section, illustrating a more complex and accurate representation of the relational structure linking animals and cages in the housing facility. :label:`cagestays`

Irregularity Management
~~~~~~~~~~~~~~~~~~~~~~~

The atomized schema seeks to introduce structure wherever possible, but also provides a bare minimum set of free-text fields, to record uncategorizable occurrences.
Such events are stored in the ``irregularities`` table, and linked by a many-to-many relationship to e.g. ``Measurement`` objects.
This not only promotes irregularity re-use, but also facilitates rudimentary manual pattern detection, and the organic design of new objects within the schema.

Plotting via BehavioPy
~~~~~~~~~~~~~~~~~~~~~~

LabbookDB provides a number of powerful data selection and processing functions, which produce consistently structured dataframes that seamlessly integrate with the BehavioPy plotting API.
The forced swim test, for instance, is a preclinically highly relevant behavioural assay :cite:`Petit-Demouliere2005`, which LabbookDB can document and evaluate.
The following example code should be executable locally, contingent on LabbookDB, example database, and example data (included in Demolog) availability:

.. code-block:: python

    import matplotlib.pyplot as plt
    from labbookdb.report.behaviour import forced_swim

    start_dates = ["2017,1,31,22,0","2016,11,24,21,30"]
    forced_swim("~/syncdata/meta.db", "tsplot",
        treatment_start_dates=start_dates
        save_df="~/fst_df.csv")
    plt.show()

The above code prompts LabbookDB to traverse the complex relational structure depicted in Figure :ref:`fstschema`, in order to join the values relevant to evaluation of the forced swim test.
``Animal`` objects are joined to ``Treatment.code`` values via their relationships to ``Cage`` and ``CageStay`` objects.
This relational structure is determined by the administration of drinking water treatments at the cage level, and thus their contingence on the presence of animals in cages at the time of the treatment.
Futher, ``Evaluation.path`` values are joined to ``Animal`` objects (via their respective relationships to ``Measurement`` objects) in order to determine where the forced swim test evaluation data is stored for every animal.
Subsequently, the annotated event tracking data is processed into desired length time bins (here, 1 minute), and immobility ratios are calculated per bin.
Finally, the data is cast into a consistent and easily readable dataframe (formatted in the same fashion as Table :ref:`fstdf`) which can be both saved to disk, or passed to the appropriate BehavioPy plotting function, to produce Figure :ref:`fst`.

.. raw:: latex

   \setlength{\tablewidth}{1.1\linewidth}

.. table:: Example of LabbookDB processed data output for the forced swim test. The format precisely matches the requirements of BehavioPy plotting functions. :label:`fstdf`

   +----+------------------+------------------+------------+
   | ID | Immobility Ratio | Interval [1 min] | Treatment  |
   +====+==================+==================+============+
   | 28 |   0.2635         | 3                | Control    |
   +----+------------------+------------------+------------+
   | 28 |   0.1440         | 2                | Control    |
   +----+------------------+------------------+------------+
   | 30 |   0.6813         | 3                | Control    |
   +----+------------------+------------------+------------+
   | 1  |   0.6251         | 6                | Fluoxetine |
   +----+------------------+------------------+------------+
   | 32 |   0.6695         | 5                | Fluoxetine |
   +----+------------------+------------------+------------+
   | 2  |   0.6498         | 6                | Fluoxetine |
   +----+------------------+------------------+------------+


.. figure:: fst.pdf

    Timecourse plot of the forced swim test performed on mice in different treatment groups |---| automatically generated by LabbookDB, using plotting bindings from BehavioPy. :label:`fst`

.. figure:: fst_schema.pdf
    :figclass: w
    :scale: 50%

    LabbookDB schema section relevant for constructing a plottable forced swim test dataframe. :label:`fstschema`

Discussion and Outlook
----------------------

Record Keeping
~~~~~~~~~~~~~~

Version tracking of database generation source code adequately addresses the main record keeping challenges at this stage of the project.
Additionally, it has a number of secondary benefits, such as providing comprehensive and up-to-date usage examples.
Not least of all, this method provides a very robust backup |---| as the database can always be rebuilt from scratch.
A very significant drawback of this approach, however, is poor scalability.

As the amount of metadata reposited in a LabbookDB database increases, the time needed for database re-generation may reach unacceptable levels.
Disk space usage, while of secondary concern, may also become an issue.
Going forward, better solutions for record keeping should be implemented.

Structure Migration
~~~~~~~~~~~~~~~~~~~

The long-term unsustainability of database source code tracking also means that a more automated means of structure migration should be developed, so that LabbookDB databases can be re-cast from older relational structures into improved and extended newer structures -
instead of relying on source code editing and regeneration from scratch.
Possibly, this could be handled by shipping an update script with every release |---| though it would be preferable if this could be done in a more dynamic, rolling release fashion.

Data Input
~~~~~~~~~~

Data input via sequential Python function calls requires a significant amount of boilerplate code, and appears very intransparent for users unaccustomed to the Python syntax.
It is also requires interfacing with an editor, minding syntax and formatting conventions, and browsing directory trees for the appropriate file in which to reposit the function calls.

While LabbookDB provides a command line interface to input the exact same data with the exact same dictionary and string conventions with arguably less boilerplate code, this input format has not been implemented for the full database generation source code.
The main concern precluding this implementation is that the syntax, though simplified form standard SQL, is not nearly simple enough to be relied on for the robustness of thousands of manual input statements generated on-site.

A better approach may be to design automated recording workflows, which prompt the researcher for values only, while applying structure internally, based on a number of templates.
Another possibility would be to write a parser for spreadsheets, which applies known LabbookDB input structures, and translates them into the internal relational representation.
This second approach would also benefit from the fact that spreadsheets are already a very popular way in which researchers record their metadata -
and could give LabbookDB the capability to import large numbers of old records, with comparatively little manual intervention.

Not least of all, the ideal outlook for LabbookDB is to automatically handle as much of the data input process as possible, e.g. via specialized sensors, via semantic image :cite:`You_2016_CVPR` or video evaluation, or via an entity-barcode-scanner (as currently used by the iRATS system)    .
This poses nontrivial engineering challenges in excess of relation modelling, and requires distinctly more manpower than currently available.
However, LabbookDB is from the licensing point of view suitable for use in commercial products, and additional manpower may be provided by science service providers interested in offering powerful, transparent, and extendable metadata tracking to their discerning customers.