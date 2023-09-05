---
layout: page
title: dynamic memory allocator
description: a project of 15-213 Computer Systems at CMU
importance: 5
category: work
---

private repo. [C/C++]

These are examples of my code design.

/--------------------------------Seglist design--------------------------------/

There are 15 explicit seglists beginning with the minimum block size
currently set at 16. All seglists are of type (block_t \*) except for the
seglist containing blocks of size 16, which is of type (m_block_t \*).
Functions used for (block_t \*) are also used for (m_block_t \*) by casting pointers.

All non-mini (blocks of size 16) free blocks are footerless. To find the
previous block in function coalesce_block, the second to last bit of each
block is set if the previous block is allocated. To know if the previous
block is a mini, the third to last bit of each block is set if so.

/----------------------------------Functions-----------------------------------/

- @brief Packs the `size`, `alloc`, 'p_alloc', and 'p_mini' of a block into a
  word suitable for use as a packed value.
  Packed values are used for both headers and footers.
  The allocation status is packed into the lowest bit of the word. The
  allocation status of the previous block is packed into the second lowest bit.
  The mini status of the previous block is packed into the third lowest bit.
- @param[in] size The size of the block being represented
- @param[in] alloc True if the block is allocated
- @param[in] p_alloc True if the previous block is allocated
- @param[in] p_mini True if the previous block is a mini
- @return The packed value

- @brief Given a payload pointer, returns a pointer to the corresponding block.
- @param[in] bp A pointer to a block's payload
- @return The corresponding block

- @brief Writes a block starting at the given address.
  This function writes both a header and footer, where the location of the footer
  is computed in relation to the header.
- @param[out] block The location to begin writing the block header
- @param[in] size The size of the new block
- @param[in] alloc The allocation status of the new block
- @param[in] p_alloc The allocation status of the previous block
- @param[in] p_mini The mini status of the previous block

- @brief Finds the next consecutive block on the heap.
  This function accesses the next block in the "implicit list" of the heap by
  adding the size of the block.
- @param[in] block A block in the heap
- @return The next consecutive block on the heap
- @pre The block is not the epilogue

- @brief Merges two free blocks if they are next to each other in the heap.
  Only uses "find_prev" if the previous block is a non-mini free block. Checks for
  this by finding the p_alloc and p_mini bits of the input block. If the previous
  block is a mini, then subtracts the minimum block size from the block.
- @param[in] block
- @return block
- @post There are no free blocks next to the block

- @brief Finds a free block for an allocation.
  Finds a mini or non-mini block. If no blocks in the seglist fit, searches the
  next seglist with larger sizes. If no seglists fit, returns NULL.
- @param[in] asize The size of the allocated block
- @return A free block with size > asize

- @brief Checks the heap for errors.
  Runs silently.
- @return False if errors in heap

- @brief Checks the seglists for errors.
  Runs silently.
- @return False if errors in explicit list

- @brief Allocates a free block.
  Searches seglists for a free block of size greater than 'size'. If no block is
  found, extends the heap. Rewrites the allocated block. Splits block if the
  unused portion of the block is greater than the minimum block size.
- @param[in] size
- @return A pointer to the payload of the allocated block

- @brief Frees an allocated block.
  Inserts the free block into seglist of size that is an upper bound of the block
  size. Coalesces if there are free blocks next to the block. Sets its allocation
  and mini status bits into the next block.
- @param[in] bp Payload of an allocated block
- @param[out] A free block

I'm a bit sentimental about this project because it was hard, i spent a lot of time, and my final grade was 99%.
I maxed out the throughput and utilization metrics but could not get the final 1%.
Nonetheless it was very fulfilling.
