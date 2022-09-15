Machine Learing pro1
Skip to content
Search or jump to…
Pull requests
Issues
Marketplace
Explore
 
@ahmadhelaly 
scikit-learn
/
scikit-learn
Public
Code
Issues
1.5k
Pull requests
604
Discussions
Actions
Projects
17
Wiki
Security
Insights
scikit-learn/maint_tools/check_pxd_in_installation.py /
@thomasjpfan
thomasjpfan STY Enables black with experimental_string_processing=true (#20412)
Latest commit 3ae7c76 on 29 Jun 2021
 History
 2 contributors
@thomasjpfan@jeremiedbb
62 lines (51 sloc)  1.94 KB

"""Utility for testing presence and usability of .pxd files in the installation
Usage:
------
python check_pxd_in_installation.py path/to/install_dir/of/scikit-learn
"""

import os
import sys
import pathlib
import tempfile
import textwrap
import subprocess


sklearn_dir = pathlib.Path(sys.argv[1])
pxd_files = list(sklearn_dir.glob("**/*.pxd"))

print("> Found pxd files:")
for pxd_file in pxd_files:
    print(" -", pxd_file)

print("\n> Trying to compile a cython extension cimporting all corresponding modules\n")
with tempfile.TemporaryDirectory() as tmpdir:
    tmpdir = pathlib.Path(tmpdir)
    # A cython test file which cimports all modules corresponding to found
    # pxd files.
    # e.g. sklearn/tree/_utils.pxd becomes `cimport sklearn.tree._utils`
    with open(tmpdir / "tst.pyx", "w") as f:
        for pxd_file in pxd_files:
            to_import = str(pxd_file.relative_to(sklearn_dir))
            to_import = to_import.replace(os.path.sep, ".")
            to_import = to_import.replace(".pxd", "")
            f.write("cimport sklearn." + to_import + "\n")

    # A basic setup file to build the test file.
    # We set the language to c++ and we use numpy.get_include() because
    # some modules require it.
    with open(tmpdir / "setup_tst.py", "w") as f:
        f.write(
            textwrap.dedent(
                """
            from distutils.core import setup
            from distutils.extension import Extension
            from Cython.Build import cythonize
            import numpy
            extensions = [Extension("tst",
                                    sources=["tst.pyx"],
                                    language="c++",
                                    include_dirs=[numpy.get_include()])]
            setup(ext_modules=cythonize(extensions))
            """
            )
        )

    subprocess.run(
        ["python", "setup_tst.py", "build_ext", "-i"], check=True, cwd=tmpdir
    )

    print("\n> Compilation succeeded !")

