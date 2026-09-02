3D Nuclear Volume Analyzer

Streamlit app for measuring 3D nuclear volume from DAPI Z-stack TIFF images.

Features

Upload multiple 2D TIFF slices as one Z-stack

Detect candidate nuclear ROIs

Use StarDist3D as a seed detector

Expand the seed to the whole bright DAPI nucleus

Guard against mask leakage outside the visible nuclear outline

Calculate nuclear volume from voxel count

Inspect segmentation overlays

Optional 3D mesh display

Export CSV

Volume formula

Volume (µm³) = voxel count × Pixel X × Pixel Y × Z spacing

The final volume calculation does not resize or downsample the image.

Repository structure

.
├── app.py
├── requirements.txt
└── README.md

Run locally

pip install -r requirements.txt
streamlit run app.py

Deploy on Streamlit Community Cloud

Create a GitHub repository.

Upload app.py, requirements.txt, and README.md to the repository root.

Open Streamlit Community Cloud.

Click Create app.

Select the GitHub repository and branch.

Set Main file path to app.py.

In Advanced settings, choose the Python version you want to use.

Click Deploy.

Streamlit will provide a *.streamlit.app URL that you can share.

Important

This app uses TensorFlow + StarDist3D and 3D TIFF data, so it can require substantial RAM and CPU.
Large analyses may exceed Streamlit Community Cloud resource limits even when deployment succeeds.

Always visually confirm that the colored segmentation mask matches the full white DAPI nucleus before using the measured volume.
