.. _ref-github-ci:

Building AppImages using GitHub Actions
===========================================

AppImages can easily be created and released directly from GitHub using GitHub's 
Continuous Integration, 'GitHub Actions'. 


.. contents:: Contents
   :local:
   :depth: 1

Producing an application directory using linuxdeploy
----------------------------------------------------

Please refer to the chapter :ref:`ref-packaging-from-source`.

For general information on linuxdeploy, see :ref:`ref-linuxdeploy`.


.. _ref-uploadtool:

Uploading the generated AppImage
--------------------------------

Once an Appimage has been generated, you want to upload it to GitHub Releases. 
The recommended method is to, first create a GitHub Build Artifact (zip) and then
release the built artifact (extracted).

.. code-block:: yaml

  name: Continuous
  on: [ push ]

  jobs:
    AppImage:
      runs-on: ubuntu-16.04
      steps:
       - uses: actions/checkout@v2

       - name: Build AppImage
         run: |
           # write the code to build your appimage here
           wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage
           chmod *.AppImage
           ./appimage*.AppImage AppDir -g
           rm ./appimage*.AppImage
           
           # move the appimage to a directory
           mkdir dist
           mv *.AppImage* dist/.

        - name: Upload artifact
          uses: actions/upload-artifact@v1.0.0
          with: 
            name: appimage-build
            path: 'dist/'

    Release:
      needs: [AppImage]
      runs-on: ubuntu-latest
      steps:
       - uses: actions/download-artifact@v1
         with:
           name: appimage-build

       - name: Detect Release type
         if: github.ref == 'refs/heads/master' && startsWith(github.ref, 'refs/tags/v') != true
         run: |
           echo "::set-env release_tag=continuous"
           
       - name: Release
         uses: marvinpinto/action-automatic-releases@latest
         if: github.ref == 'refs/heads/master' || startsWith(github.ref, 'refs/tags/v')
         with:
           prerelease: "!startsWith(github.ref, 'refs/tags/v')"
           draft: startsWith(github.ref, 'refs/tags/v')
           automatic_release_tag: ${{ env.release_tag }} || null
           title: ${{ env.release_tag }} || null
           files: |
             appimage-build
           

