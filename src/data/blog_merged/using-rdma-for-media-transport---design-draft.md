---
description: ''
pubDatetime: 2024-06-17
tags:
- Network
- RDMA
title: Using RDMA for Media Transport - Design draft
---

To start design the new library based on RDMA, we need to know the usage. This new feature is for MCM (media communication mesh), which is efficient data exchange between containers in node and cross nodes. It is built upon media transport library for its data path, when transport large video frame buffer, st2110 stack is used. So the DPDK environment is needed where the NIC should be bound to PMD (poll mode driver) and cannot be used by kernel anymore, that means the path can only be used as data path. Also a dedicated CPU is required for polling tx/rx packets. The overhead is rarely accepted by the cloud/edge environments where every computing resource costs. RDMA is more suitable here for cross node data transport since it does not require cpu intervension, and the NIC can still leverage kernel stack for control path stuff. furthermore, RDMA supports GPU memory which brings performance advantage for some media workloads.

In order to make it compatible with MTL API, the get/put api is used for tx/rx. In MTL TX, after creating the session, the user will call tx_get_frame to get the frame buffer from library and fill the contend by whether copy or write that buffer. After processing, the user should call tx_put_frame so the frame is ready to be handled by library for transmitting. whether the user can successfully get the frame for producing depends on the FPS and total framebuffer number settings and it should wait when fail until a frame is available (that is done sending by the library). In MTL RX, the user will call rx_get_frame and it will return a received frame buffer when there is one. After using the frame, the user will call rx_put_frame and the buffer can be reused by the library for next receiving.

For the new RDMA based API, we extend it a little. the data transported can not only be video/audio frame but any data type. Here is the buffer manipulating API:

```c
/**
 * Get one TX buffer from the TX session.
 * Call mtl_rdma_tx_put_buffer to return the buffer to session.
 *
 * @param handle
 *   The handle to the TX session.
 * @return
 *   - NULL if no available buffer in the session.
 *   - Otherwise, the buffer pointer.
 */
struct mtl_rdma_buffer* mtl_rdma_tx_get_buffer(mtl_rdma_tx_handle handle);

/**
 * Put back the buffer which get by mtl_rdma_tx_get_buffer to the TX
 * session.
 *
 * @param handle
 *   The handle to the TX session.
 * @param buffer
 *   the buffer pointer by mtl_rdma_tx_get_buffer.
 * @return
 *   - 0 if successful.
 *   - <0: Error code if put fail.
 */
int mtl_rdma_tx_put_buffer(mtl_rdma_tx_handle handle, struct mtl_rdma_buffer* buffer);

/**
 * Get one RX buffer from the RX session.
 * Call mtl_rdma_rx_put_buffer to return the buffer to session.
 *
 * @param handle
 *   The handle to the RX session.
 * @return
 *   - NULL if no available buffer in the session.
 *   - Otherwise, the buffer pointer.
 */
struct mtl_rdma_buffer* mtl_rdma_rx_get_buffer(mtl_rdma_rx_handle handle);

/**
 * Put back the buffer which get by mtl_rdma_rx_get_buffer to the RX
 * session.
 *
 * @param handle
 *   The handle to the RX session.
 * @param buffer
 *   the buffer pointer by mtl_rdma_rx_get_buffer.
 * @return
 *   - 0 if successful.
 *   - <0: Error code if put fail.
 */
int mtl_rdma_rx_put_buffer(mtl_rdma_rx_handle handle, struct mtl_rdma_buffer* buffer);
```

The buffer is defined as a continuing memory block:

```c
/** The structure info for buffer meta. */
struct mtl_rdma_buffer {
  /** Buffer address, immutable at runtime */
  void* addr;
  /** Buffer data capacity, immutable at runtime */
  size_t capacity;
  /** Buffer valid data offset, mutable at runtime */
  size_t offset;
  /** Buffer valid data size, mutable at runtime */
  size_t size;
  /** Buffer sequence number */
  uint32_t seq_num;
  /** Buffer timestamp, use nanoseconds in lib */
  uint64_t timestamp;

  /** User metadata */
  void* user_meta;
  /** User metadata size */
  size_t user_meta_size;
};
```

we also reserve the usermeta info so the user can fill any type of metadata in it and transport it along with the buffer.

Below is a figure of the API design.