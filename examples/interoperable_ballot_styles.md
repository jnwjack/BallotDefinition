# Interoperable Ballot Style Validator

## Overview

The interoperable ballot style validator is intended to provide a method for vendors and other producers of ballots to verify that the NIST ballot definition (BD) file associated with one or more ballot styles visually matches the target areas appearing on ballot proof PDFs. It translates XML Ballot Definition CDF data into [XFDF](https://www.pdfa.org/resource/iso-19444-xfdf/) containing annotations that are overlayed on a PDF of a ballot to allow visual inspections to be performed. Just as Mylar overlays used by voting systems vendors ensure ballots meet production requirements, this tool ensures ballot proofs meet requirements for interoperable ballot styles.

Three ballot styles are represented as examples. PDFs that reproduce ballot styles used by ES&S and Hart Intercivic are included as well as a more comprehensive example using the [NIST Ballot Definition Prototype](../Ballot_Definition_Prototype.pdf). The vendor-specific ballots include only presidential contests in their ballot definition files while the prototype ballot includes all contests appearing on the ballot.

> Any mention of commercial products is for information only; it does not imply recommendation or endorsement by NIST.

While detailed documentation of the code will not be provided here, an XSLT file that can be used to transform custom ballot definition XML files into XFDF is included and instructions for executing it using SaxonJ follow at the [end of this document](#creating-custom-ballot-definition-overlays).

## Requirements

- Adobe Acrobat Reader DC (or another PDF viewer that supports XFDF annotations)
- (Optional) A text editor that can render properly-formatted XML files. A code editor with linting capabilities is recommended.

## Viewing the Example Ballots

The files provided in this repository represent ballots produced by Hart (`physical_ballot_def_1*`), ES&S (`physical_ballot_def_2*`) voting systems and as part of the [Ballot Definition Prototype](../Ballot_Definition_Prototype.pdf), (`physical_ballot_def_3*`). Adobe Acrobat Reader or equivalent is required to view them and in order to render the XFDF annotations that overlay the ballot image.

In order to view the annotation overlay, you may open the XFDF files directly. Each XFDF has a reference to the PDF it is intended to overlay, and Acrobat will automatically bring up the PDF and create the annotations.

Target areas defined in the XML ballot definitions should be outlined in red and overlap those on the ballot. Mousing over a target area will display a snippet of ballot definition XML that is associated with it. This can also be viewed in the side panel by selecting View -> Tools -> Comment -> Open from the command bar.

## Ballot Definitions

The visual format and features of a ballot are defined in a ballot definition XML file. Each of the PDFs is associated with a similarly-named XML file that describes it. In general, the files define four major sets of data:

1. The format of a ballot,
2. Information about contests, such selection and rules,
3. The structure of a single ballot style, and
4. Metadata about the ballot such as issuer and date of election.

> The sections below reference fragments of `physical_ballot_def_2.xml`.

### The `BallotFormat` Element

The layout of a ballot is defined at the top of the file within the `BallotFormat` element. The dimensions of the ballot are expressed in points (`pt`).

```xml
<BallotFormat ObjectId="bf-ess">
	<ExternalIdentifier>
		<Type>local-level</Type>
		<Value>E1</Value>
	</ExternalIdentifier>
	...
	<LongEdge>1080</LongEdge>
	...
	<MeasurementUnit>pt</MeasurementUnit>
	<Orientation>portrait</Orientation>
	<ReadMethod>omr</ReadMethod>
	<ShortEdge>612</ShortEdge>
</BallotFormat>
```

Information about fiducial marks, such as those shown on the page borders of `*def_2.pdf`, can also be included. The `GeometryId` property is used in elements to define the size and shape of target areas with a shared ID.

```xml
<FiducialMark>
	<H>8.5</H>
	<Sheet>1</Sheet>
	<Side>front</Side>
	<W>11.5</W>
	<X>39.5</X>
	<Y>95.6</Y>
	<GeometryId>g-2</GeometryId>
</FiducialMark>
```

Information about the location and dimensions of the symbology used to store ballot style identification information on the ballot is defined.
	
```xml
<mCDFArea>
	<H>100</H>
	<Sheet>1</Sheet>
	<Side>front</Side>
	<W>100</W>
	<X>125</X>
	<Y>125</Y>
	<Symbology>QR_CODE</Symbology>
</mCDFArea>
```
> The example ballot for ES&S does not use the mCDF, but an mCDFArea has been created to illustrate its use.

### The `BallotStyle` Element

The `BallotStyle` element follows the `BallotFormat` element. Its child elements describe various physical features of the ballot.

The `PhysicalContestOption` element contains information about individual target areas, such as their position and dimensions as well as the geometric indicator associated with each. The `IndicatorId` here serves the same purpose as the `GeometryId` above.

```xml
<PhysicalContestOption>
	<ContestOptionId>cs-biden-harris</ContestOptionId>
	<OptionPosition>
		<H>13.5</H>
		<Sheet>1</Sheet>
		<Side>front</Side>
		<W>25</W>
		<X>224.25</X>
		<Y>263.25</Y>
		<IndicatorId>g-1</IndicatorId>
		<NumberVotes>1</NumberVotes>
	</OptionPosition>
</PhysicalContestOption>
```

### The `Candidate` and `Contest` Elements

The `Candidate` element contains information related to candidates and used to defined contest selections. `Candidates` have IDs and their name as it appears on the ballot defined in this section. The IDs link candidates and the contests with which they are associated.

```xml
<Candidate ObjectId="can-kamala-harris">
	<BallotName>
		<Text Language="en">Kamala D. Harris</Text>
	</BallotName>
</Candidate>
```

The `Contest` element includes information about each contest and the contest selections associated with it. It also defines the rules applied to each contest. In the example below, one of the values assigned to the `CandidateIds` element corresponds to the Candidate ID defined above.

```xml
<Contest ObjectId="cc-president" xsi:type="CandidateContest">
	<ContestOption xsi:type="CandidateOption" ObjectId="cs-biden-harris">
		<CandidateIds>can-joseph-biden can-kamala-harris</CandidateIds>
	</ContestOption>
	<ContestOption xsi:type="CandidateOption" ObjectId="cs-hawkins-walker">
		<CandidateIds>can-howie-hawkins can-angela-walker</CandidateIds>
	</ContestOption>
	<ContestOption xsi:type="CandidateOption" ObjectId="cs-jorgensen-cohen">
		<CandidateIds>can-jo-jorgensen can-spike-cohen</CandidateIds>
	</ContestOption>
	<ContestOption xsi:type="CandidateOption" ObjectId="cs-trump-pence">
		<CandidateIds>can-donald-trump can-michael-pence</CandidateIds>
	</ContestOption>
	<ElectionDistrictId>gpu-ohio</ElectionDistrictId>
	<Name>PRESIDENT OF THE UNITED STATES</Name>
	<VoteVariation>n-of-m</VoteVariation>
	<NumberElected>1</NumberElected>
	<VotesAllowed>1</VotesAllowed>
</Contest>
```

### Additional Definitions and Metadata

Properties associated with the `GeometryId` and `IndicatorId` are defined in the `Geometry` element. They include shape and line width.

```xml
<Geometry ObjectId="g-1">
	<ShapeType>ellipse</ShapeType>
	<StrokeWidth>0.33</StrokeWidth>
</Geometry>
```

Metadata about the ballot issuer and related information follows at the end of the file.

```xml
<Issuer>NIST POC</Issuer>
<IssuerAbbreviation>NIST</IssuerAbbreviation>
<SequenceStart>1</SequenceStart>
<SequenceEnd>1</SequenceEnd>
<VendorApplicationId>Hand Written</VendorApplicationId>
<Version>1.0.0</Version>
```

## Creating Custom Ballot Definition Overlays

The file `bd_to_xfdf.xlst` can be used to create overlays from custom ballot definition XML files. To run the XSLT template with a custom file, a XSLT 3.0 engine is required. The example below requires the use of:

- [Java SE 8 (JDK 1.8) or later](https://developers.redhat.com/products/openjdk/overview)
- [SaxonJ Home Edition 11](https://sourceforge.net/projects/saxon/files/Saxon-HE/11/) or later (frameworks other than Java are supported, but this document assumes the use of the Java version)

1. Navigate to the `SaxonHE11-4J` folder contained in the extracted ZIP file downloaded from the link above.
2. In a terminal window or command prompt, execute the following command, where `<template>` is the path to the XSLT file in this repo, `<definition>` is the ballot definition file, `<bf_local_level_id>` is the `local-level` `ExternalIdentifier` for the target `BalltoFormat`, and `<bs_local_level_id>` is the `local-level` `ExternalIdentifier` for the target `BallotStyle`:

```bash
java -jar './saxon-he-11.4.jar' -s:<definition> -xsl:<template> -o:<output_file> targetBallotFormat=<bf_local_level_id> targetBallotStyle=<bs_local_level_id>
```

NB: If a different kind of `ExternalIdentifier` type is used (e.g. `local-level`, `state-level`, etc.) then you may provide that type using the additional optional parameters `targetBallotFormatType` and `targetBallotStyleType`.

Limitations:

- Only Ballot Definition XML instances are currently supported.
- Only ballots using a ballot format with `Orientation = 'portrait'` are currently supported.
- Only ballots using a ballot format with `MeasurementUnit = 'pt'` are currently supported.
