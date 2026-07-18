# Hardware-Accelerated Image Scaler & Spatial Filter

## Overview
This repository contains a Verilog-based, hardware-accelerated image scaling engine. The design implements Bilinear Interpolation for dynamic image resizing and integrates a 3x3 Convolutional Spatial Filter (Gaussian approximation) to prevent aliasing artifacts during downscaling operations. 

The architecture is highly optimized for silicon area and power, utilizing fixed-point arithmetic (UQ0.8 format) and multiplier-less filter structures. It is heavily parameterized, allowing for compile-time configuration of input/output resolutions and channel depths (e.g., Grayscale or RGB).

## Key Features

Bilinear Interpolation: High-quality spatial scaling using weighted average pixel computation.

Anti-Aliasing Filter: An integrated 3x3 low-pass filter specifically active during downscaling. It utilizes hardware-efficient bit-shifts (<< 4, << 5, << 6) to replace DSP slice multipliers, forming a perfectly normalized 256-weight matrix.

Custom Non-Restoring Divider: Dedicated hardware division module to accurately calculate fractional stepping ratios (win_by_wout, hin_by_hout) between input and output dimensions.

Dynamic Coordinate Generation: Automated X and Y traversal logic managed by a dedicated coordinate generator module.

Area-Efficient Fixed-Point Math: All fractional scaling weights are computed using 8-bit precision (0.8 fixed-point), ensuring efficient accumulation without floating-point overhead.

## Architecture & Module Hierarchy

The system is orchestrated by an internal Finite State Machine (FSM) that controls division, filtering, interpolation, and write-back phases.

top.v (or your top module name): The core FSM and data path. It instantiates the sub-modules, handles memory indexing, applies the 3x3 spatial filter, and calculates the final bilinear interpolation using an accumulator.

div.v: Implements a non-restoring division algorithm. It calculates the base coordinates and the fractional weights required for interpolation.

coordgen.v: Generates and tracks the current output pixel coordinates (x_out, y_out). It is controlled via step-and-reset signals driven by the top-level FSM.

## State Machine Pipeline

INIT & Division Phase: Calculates horizontal and vertical scaling ratios.

Blur Phase: If downscaling, applies the 3x3 spatial shift-and-add filter.

Accumulation Phase: Sequentially computes the four weighted pixel values for bilinear interpolation to avoid massive simultaneous multiplier instantiation.

Write-back Phase: Normalizes the accumulated 26-bit register back to standard 8-bit color depth and manages loop boundaries.

## Simulation & Testing Workflow

To verify the hardware logic and visually evaluate the scaling accuracy across different ratios, this project utilizes an automated end-to-end testing pipeline combining Python and Verilog.

Pre-Processing (Python): A Python script reads standard image files, extracts the raw pixel data, and flattens it into a hexadecimal format (.hex).

Hardware Simulation (Verilog): The Verilog testbench uses the $readmemh command to load the pre-processed .hex data into the internal memory arrays. The FSM processes the image and outputs the scaled results to a new text file using $writememh.

Post-Processing & Visual Verification (Python): A secondary Python script parses the output .hex file, reconstructs the image matrix based on the configured dimensions, and saves it as a standard viewable image. This allows for direct visual comparison of the hardware's bilinear interpolation and spatial filtering against expected software models.
