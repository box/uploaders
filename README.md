# Verold Studio Uploader API

The Uploader API allows for developers to create integrations between modelling tools, ​3D scanning software, or other sources of 3D content. You can easily upload models and textures to Verold Studio through the API.

## POST projects.json:

The method POST on http://studio.verold.com/projects.json allows you to create a new project ​with one or more assets (models, textures, materials, etc.). An example of using the API is given here, with detailed descriptions of the parameters below.

    # easy_install poster
    from poster.encode import multipart_encode
    from poster.streaminghttp import register_openers
    import urllib2
 
    # Register the streaming http handlers with urllib2
    register_openers()
 
    import json
    import base64
    import urllib2
 
    url="http://studio.verold.com/projects.json"
 
    data = {
      'api_key': "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      'title': "My First Project",
      'description': "Hello World!  This is my first project!",
      'model': open("./model.zip"),
      'tags': "test,first",
      'private': "public"
    }
 
    datamulti, headers = multipart_encode(data)
 
    request = urllib2.Request(url, datamulti, headers)
    print urllib2.urlopen(request).read()
    
    
### api_key
The user’s API key, used for authentication. You can get your personal API key from here.

### model
The model file. For best results, we recommend uploading a ZIP file containing the model (in OBJ, Collada, or FBX format) and all textures.

### title (optional)
The project title.

### description (optional)
The project description.

### tags (optional)
A comma-separated list of project tags.

### private (optional)
Set to “public” or “private” (Pro accounts only).  If private is set to “private” for a non-Pro account, the value is silently changed to “public” without generating an error. Defaults to private for Pro accounts.

### scale_to_size (optional)
Scales the model(s) to fit in a cube of this size, placed on the ground plane. Default: don’t scale model(s).

### merge_models (optional)
If “1”, treat multiple models as a single model.  When uploading multiple models in a ZIP archive, it is often best to set merge_models to “1” so that the relative sizing of each model is preserved; otherwise, models are scaled independently according to scale_to_size. Default: Don’t merge models.    

## Response

If the project is created successfully, the response code is 201 and the response takes the form:

    { "id":"511399f3fc77b30200000469", … }

The project can be viewed at:

> http://studio.verold.com/projects/511399f3fc77b30200000469

If an error occurs, check the response code.  For example, an invalid API key results in a 401 (unauthorized) response.

# Determining Processing Success/Failure

When a model is uploaded to Verold Studio, it is passed off asynchrnously to a cluster of processing servers where the models are optimized, compressed, and added to your project scene. This means, that if you redirect the user directly to Verold Studio immediately after the model files have been uploaded, that the user will likely get there before the modelrs are ready in the scene. Verold Studio will detect that there are jobs processing, and inform the user that a model is on the way. When it's ready, it will be automatically loaded into the scene.

We deliberately do not hold the request open while the uploaded models are processing. However, if you would like to hold the user in your tools until the models have been fully processed, you can easily achieve this by polling on the jobs queue of the project. Then you can only redirect the user to Verold Studio after the model has been fully processed. For this, use the Jobs API:

    GET /projects/{project_id}/jobs.json

an example url is:

    http://studio.verold.com/projects/51504881c1f95e02000002da/jobs.json

The JSON output from that link is:

    [
      {
        id: "51504899c1f95e02000002e2",
        userId: "xxxxxxxxxxxxxxxxxxxxxxx",
        projectId: "123123123123123123123",
        filePath: "uploads/asahashashgashahg/hashjahjashsahjashjashjashjahjashjsa/fighter.fbx",
        state: "COMPLETE",
        autoInstance: false,
        dateCreated: "2013-03-25T12:52:41.682Z",
        dateModified: "2013-03-25T12:49:36.931Z"
      },
      ...
    ]


There will be one entry for each uploaded file, and the state of each entry will be one of:

    'CREATED', 'STARTED', 'COMPLETE', 'ERROR'

In normal circumstances the job should go to either complete or error fairly quickly, depending on the complexity of the model.

# Optimization

The Verold Uploader supports decimation and optimization during the upload process. This is especially useful for high-res 3D scans; using the optimization parameters you can very easily generate a version of your model that can be displayed in realtime. Be aware that using these optimization parameters will slow the upload process. 

For an example of how to use the optimization parameters, see the Python upload script:

    Python/VeroldUploader.py
    
You can run this as follows:

    python VeroldUploader.py -k api_key -d 0.1 -t 50000 file.obj
    
The first step of optimization is to decimate the model. You can speficy either -d for the decimation percentage, or -t for the maximum number of polygons. Models less than 20K polys load extremely fast, then up to 200K polys load pretty well. Beyond 200K polys, model loading gets slower, sometimes unbearably slow on lower powered client machines.
