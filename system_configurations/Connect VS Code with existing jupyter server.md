
# Connect to an Existing or Remote Jupyter Server
If you already have a Jupyter server running locally in your terminal or on a remote cloud machine, you can connect VS Code to that specific instance. [6]

   1. Get the Server URL: When you run jupyter notebook or jupyter lab in a terminal, it outputs a URL containing a security token. Copy that URL (it usually looks like http://localhost:8888/?token=...).
   2. In VS Code, open an existing .ipynb notebook file.
   3. Click the Kernel Picker button in the top-right corner.
   4. Choose Existing Jupyter Server.
   5. Select Enter the URL of the running Jupyter server.
   6. Paste the full URL (including the ?token=... parameter) and press Enter. [4, 6, 7, 8] 

VS Code will now offload the code computation to your pre-existing server instance. [6]
------------------------------

## Reference

[1] [https://www.youtube.com](https://www.youtube.com/watch?v=suAkMeWJ1yE)
[2] [https://www.youtube.com](https://www.youtube.com/watch?v=XtNNjFtxE4w)
[3] [https://medium.com](https://medium.com/@CompXBio/data-science-series-1-attach-running-jupyter-notebook-lab-kernel-to-vscode-a2297a820ed3)
[4] [https://ilum.cloud](https://ilum.cloud/docs/user-guides/vscode-jupyter-integration/)
[5] [https://www.youtube.com](https://www.youtube.com/watch?v=K0B2P1Zpdqs&t=370)
[6] [https://code.visualstudio.com](https://code.visualstudio.com/docs/datascience/jupyter-notebooks)
[7] [https://docs.verda.com](https://docs.verda.com/cpu-and-gpu-instances/connecting-to-jupyter-notebook-with-vs-code/)
[8] [https://www.spatialnasir.com](https://www.spatialnasir.com/2024/03/how-to-run-jupyter-notebook-in-visual.html)
