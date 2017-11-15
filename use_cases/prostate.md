# Segmentations and measurements from prostate MRI

## Summary of the use case

In this use case we will summarize the approach to encode segmentations of various structures and measurements derived using those segmentations from multi-parametric Magnetic Resonance Imaging (MRI) of the prostate.

More specifically, we will discuss encoding of the imaging-derived data discussed in the following paper:

> Fedorov A, Vangel MG, Tempany CM, Fennessy FM. Multiparametric Magnetic Resonance Imaging of the Prostate: Repeatability of Volume and Apparent Diffusion Coefficient Quantification. _Investigative Radiology_; 2017;52(9):538–546 http://dx.doi.org/10.1097/RLI.0000000000000382.

The imaging-derived data discussed in that paper consists of the two components:

1. Segmentations of the following structures:
 * Whole gland of the prostate
 * Peripheral zone of the prostate
 * Suspected tumor area of the prostate within the peripheral zone of the prostate
 * Normal-appearing area of the peripheral zone

2. Segmentation-based measurements: the manuscript is concerned with evaluating the repeatability of the
 * mean volume of the regions listed above, and
 * mean Apparent Diffusion Coefficient (ADC) values calculated over the segmentation-defined regions.

## Conversion to DICOM SEG

The first convenient place to start with generating JSON files for the `dcmqi` SEG converter is always http://qiicr.org/dcmqi/#/seg.

The most confusing part is typically how to find the codes to encode specific items. In our case, some structures have codes already included in the DICOM standard, while some other codes had to be looked up in SNOMED.

Make sure to select DICOM master list in the "Segmentation Category Type Context Name". This will give you access to all of the codes in the standard. 

* Whole prostate gland: we encode anatomic structure (short selection list in the Category selector), and can quickly locate the code for the prostate:
```
        "SegmentedPropertyCategoryCodeSequence": {
          "CodeValue": "T-D000A",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Anatomical Structure"
        },
        "SegmentedPropertyTypeCodeSequence": {
          "CodeValue": "T-9200B",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Prostate"
        }
```
* Peripheral zone: this one is more difficult - no code could be located. We needed to consult http://browser.ihtsdotools.org/ to find "Peripheral zone of the prostate", which has SCTID 279706003. Next, we looked up SRT code for SCTID in an older version of SNOMED (this was done by our good friend David Clunie!). Result:
```
        "SegmentedPropertyCategoryCodeSequence": {
          "CodeValue": "T-D000A",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Anatomical Structure"
        },
        "SegmentedPropertyTypeCodeSequence": {
          "CodeValue": "T-D05E4",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Peripheral zone of the prostate"
        }
```
* Suspected tumor tissue: the most suitable code is "Lesion", but it is not available in the web interface we used before ... We will need to fix this... Note that for "Morphologically altered structure" we can (and should!) also specify "AnatomicRegionSequence" to encode the location of the lesion. In our case, all of the lesions are in the peripheral zone of the prostate.
```
       "SegmentedPropertyCategoryCodeSequence": {
          "CodeValue": "M-01000",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Morphologically Altered Structure"
        },
        "SegmentedPropertyTypeCodeSequence": {
          "CodeValue": "M-01100",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Lesion"
        },
        "AnatomicRegionSequence": {
          "CodeValue": "T-D05E4",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Peripheral zone of the prostate"
        }
```
* Normal tissue: here we can use generic "Tissue" for category, and a special code (which again is not available in our web interface) for Normal tissue:
```
       "SegmentedPropertyCategoryCodeSequence": {
          "CodeValue": "M-01000",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Morphologically Altered Structure"
        },
        "SegmentedPropertyTypeCodeSequence": {
          "CodeValue": "G-A460",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Normal"
        },
        "AnatomicRegionSequence": {
          "CodeValue": "T-D05E4",
          "CodingSchemeDesignator": "SRT",
          "CodeMeaning": "Peripheral zone of the prostate"
        }
```

Other attributes are rather trivial to populate:
* `BodyPartExamined` should be `PROSTATE` (all caps is important! ... we should add explanation to this ...)
* `SegmentDescription` can be populated with an abbreviated name reflecting the structure being segmented (note: this attribute can be at most 64 characters long!)
* `SegmentAlgorithmName` should be set to `Slicer`
* `SegmentAlgorithmType` to `MANUAL`, since that is how segmentations were created
