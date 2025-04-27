
<a id="orgbcc99b2"></a>

# Orfeo Toolbox, a library for satellite image processing

Although orfeo toolbox is one of the most complete satellite image processing libraries on the market, I am surprised by how little use or knowledge of the tool it has outside the French space agency (CNES) and the ecosystem of companies, centered in Toulouse, and some Italian companies.

OTB is an open source tool, both in the literal sense, i.e. the source code is available <sup><a id="fnr.1" class="footref" href="#fn.1" role="doc-backlink">1</a></sup>, as well as in the broadest sense and that is that it follows the philosophy of open source collaborative development, <sup><a id="fnr.2" class="footref" href="#fn.2" role="doc-backlink">2</a></sup>  forgive the redundancy. 

 The capabilities of the library are varied, and in my opinion allow to have a quite precise control over which operations you want to apply on a satellite image. The main abstractions of the library are inherited from a medical image processing library, called ITK, written and developed in C++.<sup><a id="fnr.3" class="footref" href="#fn.3" role="doc-backlink">3</a></sup> . This allows, depending on the level of abstraction the user is interested in, to write processing algorithms from scratch in C++, or to benefit from the Python portability of the library.
 
In my opinion, the most difficult part of OTB, is its installation, its developers for the sake of flexibility and the desire to compile only the necessary tools, has made the installation via compilation cumbersome. Obviously work has been done on this, and mechanisms such as *superbuild* have helped to improve the user experience when installing the library.

Another weakness is the need to recompile the entire library to compile for a specific python version, which overly complicates the use of this tool as a dependency in any python-based project, which is now almost a standard in the remote sensing industry.

 My favorite way to overcome the above problems is to simply install OTB, using the build made by the Iota-2 development team.<sup><a id="fnr.4" class="footref" href="#fn.4" role="doc-backlink">4</a></sup>. This allows you to generate a python environment with minimal dependencies, which allows you to use OTB instantly. Doing so is as easy as executing the following lines <sup><a id="fnr.5" class="footref" href="#fn.5" role="doc-backlink">5</a></sup> :

    # create an empty micromamba environment: otb-env
    micromamba create --name otb-env
    # enable it
    micromamba activate otb-env
    # install recent python 3.11
    micromamba install python=3.11
    # install mamba
    micromamba install mamba
    # install otb
    mamba install otb -c iota2-2
    # disable environment and reload it
    micromamba deactivate
    micromamba activate otb-env

To verify that the installation has been successful, you can run the following lines in a python repl, once you have verified that you are pointing to the correct python repl.

    import otbApplication as otb
    
    print(otb.Registry_GetAvailableApplications())
    
    print(len(otb.Registry_GetAvailableApplications()))

As you can see from the list above, there are 118 otb applications installed, this allows us to do a lot of things !!!!

To keep things simple, we can simply read, the basic information of an image, using the otb application *ReadImageInfo*, an advantage of using this application, is that we can get similar information with the *gdalinfo* application and validate that the information is correct.
    
    image_path = "~/Documents/datasets/Venus/VENUS-XS_20201030-114859-000_L2A_BAMBENW2_C_V3-0/VENUS-XS_20201030-114859-000_L2A_BAMBENW2_C_V3-0_FRE_B2.tif"
    
    import otbApplication as otb
    
    # Let's create the application with codename "Smoothing"
    app = otb.Registry.CreateApplication("ReadImageInfo")
    
    # We set its parameters
    app.SetParameterString("in", image_path)
    
    # This will execute the application and save the output file
    app.ExecuteAndWriteOutput()

 You can obtain with the above code information like the following:

To perform a comparison it is possible to use a gdal command, like the following:

    gdalinfo ~/Documents/datasets/Venus/VENUS-XS_20201030-114859-000_L2A_BAMBENW2_C_V3-0/VENUS-XS_20201030-114859-000_L2A_BAMBENW2_C_V3-0_FRE_B2.tif

In turn, you get an output like the following:

Of course, all this is an eye opener, the library allows to perform many many tasks, such as mosaics, optical calibration, rasterization, feature extraction, radiometric indices, supervized and non-supervized classifications, radar image processing, object-based image processing (segmentation), 3D processing, among others.

From this small blog, I encourage you to try the library and to exploit it in your personal and industrial projects.


# Footnotes

<sup><a id="fn.1" href="#fnr.1">1</a></sup> [Main Repositories / otb · GitLab](https://gitlab.orfeo-toolbox.org/orfeotoolbox/otb)

<sup><a id="fn.2" href="#fnr.2">2</a></sup> [Código abierto - Wikipedia, la enciclopedia libre](https://es.wikipedia.org/wiki/C%C3%B3digo_abierto)

<sup><a id="fn.3" href="#fnr.3">3</a></sup> <https://itk.org/>

<sup><a id="fn.4" href="#fnr.4">4</a></sup> <https://docs.iota2.net/master/>

<sup><a id="fn.5" href="#fnr.5">5</a></sup> Ejemplo modificado de la documentacion de Iota-2

