# Code Optimization Project Overview

This directory contains detailed notes and study materials for the course **Code Optimization in Production Compilers** at MFF UK. The primary goal of this repository is to create a comprehensive set of technical notes focused on the internals of **GCC**.

## Project Goals

- **Comprehensive Notes**: Create detailed markdown notes for each topic in the syllabus.
- **Technical Depth**: Ensure technical details are not omitted. Include examples, counter-examples, explanations, pseudocode, and mathematical notation.
- **OCR Transcription**: Transcribe blackboard images from the `blackboards` directory into structured text to serve as a basis for the notes.

## Syllabus

1.  **Existing Free Compilers**: GCC, LLVM, Open64, etc.
2.  **Intermediate Code Representation**: Program representation in IR.
3.  **Control Flow Graph (CFG)**.
4.  **SSA Form**.
5.  **Basic Optimizations on SSA Form**: Constant propagation, Global Value Numbering, etc.
6.  **Alias Analysis and Memory Manipulation Optimization**.
7.  **Loop Detection and Optimization**.
8.  **Optimization for Memory Hierarchy**.
9.  **Interprocedural Optimization**: Translation organization, inlining, constant propagation, etc.
10. **Profile-Driven Optimization & Dynamic Optimization**.

## Project Structure

- **notes/**: The primary directory containing markdown files with technical notes.
- **blackboards/**: A symlink to external storage containing lecture blackboard images.
- **transcriptions/**: (Planned) Directory for OCR-transcribed text from blackboard images, organized by lecture.

## Workflow

1.  **OCR Transcription**:
    *   Transcribe images from `blackboards/` into a new directory.
    *   Naming convention: `transcriptions/<lecture_number>/<original_image_name>.txt`.
2.  **Note Creation**:
    *   Generate one markdown note per syllabus topic.
    *   Synthesize information from existing notes, transcriptions, and GCC documentation.
    *   Prioritize technical precision and illustrative examples.
