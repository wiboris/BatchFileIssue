Issue: Failing to be able to define a file stream:

In orginal swagger (BatchSwaggerFile\BatchService.json) the File_GetFromComputeNode operation states that it produces the following:


        "produces": [
          "application/json",
          "application/octet-stream"
        ],

And in the response body it returns an object of type file

            "description": "The file content.",
            "schema": {
              "type": "object",
              "format": "file"
            }

And when the python SDK is produced from autorest using the swagger (SwaggerPythongSDKVersion\_file_operation.py) the operation method get_from_compute_node() takes the response and uses a Stream_download operation. Note that "stream=True" in the pipleline


        pipeline_response = self._client._pipeline.run(  # type: ignore # pylint: disable=protected-access
            request, stream=True, **kwargs
        )

        response = pipeline_response.http_response

        if response.status_code not in [200]:
            map_error(status_code=response.status_code, response=response, error_map=error_map)
            error = self._deserialize.failsafe_deserialize(_models.BatchError, pipeline_response)
            raise HttpResponseError(response=response, model=error)

        response_headers = {}
        response_headers["client-request-id"] = self._deserialize("str", response.headers.get("client-request-id"))
        response_headers["request-id"] = self._deserialize("str", response.headers.get("request-id"))
        response_headers["ETag"] = self._deserialize("str", response.headers.get("ETag"))
        response_headers["Last-Modified"] = self._deserialize("rfc-1123", response.headers.get("Last-Modified"))
        response_headers["ocp-creation-time"] = self._deserialize("rfc-1123", response.headers.get("ocp-creation-time"))
        response_headers["ocp-batch-file-isdirectory"] = self._deserialize(
            "bool", response.headers.get("ocp-batch-file-isdirectory")
        )
        response_headers["ocp-batch-file-url"] = self._deserialize("str", response.headers.get("ocp-batch-file-url"))
        response_headers["ocp-batch-file-mode"] = self._deserialize("str", response.headers.get("ocp-batch-file-mode"))
        response_headers["Content-Type"] = self._deserialize("str", response.headers.get("Content-Type"))
        response_headers["Content-Length"] = self._deserialize("int", response.headers.get("Content-Length"))

        deserialized = response.stream_download(self._client._pipeline)


With CADL I was not able to define the response body in the same way the original swagger, I tried the following, See main.cadl  

      @body
      @doc("A response containing the file content.")
      file: bytes;

      or

      @body
      @doc("A response containing the file content.")
      file: object

In both case the resulting python sdk generated takes the response and does a _deserialize on it. Note that "stream=false" in the pipeline.

        pipeline_response: PipelineResponse = self._client._pipeline.run(  # pylint: disable=protected-access
            request, stream=False, **kwargs
        )

        response = pipeline_response.http_response

        if response.status_code not in [200]:
            map_error(status_code=response.status_code, response=response, error_map=error_map)
            raise HttpResponseError(response=response)

        response_headers = {}
        response_headers["client-request-id"] = self._deserialize("str", response.headers.get("client-request-id"))
        response_headers["request-id"] = self._deserialize("str", response.headers.get("request-id"))
        response_headers["etag"] = self._deserialize("str", response.headers.get("etag"))
        response_headers["last-modified"] = self._deserialize("str", response.headers.get("last-modified"))
        response_headers["ocp-creation-time"] = self._deserialize("str", response.headers.get("ocp-creation-time"))
        response_headers["ocp-batch-file-isdirectory"] = self._deserialize(
            "bool", response.headers.get("ocp-batch-file-isdirectory")
        )
        response_headers["ocp-batch-file-url"] = self._deserialize("str", response.headers.get("ocp-batch-file-url"))
        response_headers["ocp-batch-file-mode"] = self._deserialize("str", response.headers.get("ocp-batch-file-mode"))
        response_headers["content-length"] = self._deserialize("int", response.headers.get("content-length"))

        deserialized = _deserialize(bytes, response.json())