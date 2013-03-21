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
