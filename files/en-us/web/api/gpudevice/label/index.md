---
title: "GPUDevice: label property"
short-title: label
slug: Web/API/GPUDevice/label
page-type: web-api-instance-property
browser-compat: api.GPUDevice.label
---

{{APIRef("WebGPU API")}}{{SecureContext_Header}}{{AvailableInWorkers}}

The **`label`** read-only property of the
{{domxref("GPUDevice")}} interface is a string providing a label that can be used to identify the object, for example in {{domxref("GPUError")}} messages or console warnings.

## Value

A string. If no label value has previously been set, getting the label returns an empty string.

## Examples

```js
async function init() {
  if (!navigator.gpu) {
    throw Error("WebGPU not supported.");
  }

  const adapter = await navigator.gpu.requestAdapter();
  if (!adapter) {
    throw Error("Couldn't request WebGPU adapter.");
  }

  // Create a GPUDevice
  const device = await adapter.requestDevice();

  // Set a label
  device.label = "my_label";

  // Get a label
  console.log(device.label);

  // …
}
```

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}

## See also

- The [WebGPU API](/en-US/docs/Web/API/WebGPU_API)
