---
layout: page
title: dynamic memory allocator
description: a project of 15-213 Computer Systems at CMU
importance: 5
category: work
---

private repo. [C/C++]

This is my code sample.

/--------------------------------Seglist design--------------------------------/

There are 15 explicit seglists beginning with the minimum block size
currently set at 16. All seglists are of type (block_t \*) except for the
seglist containing blocks of size 16, which is of type (m_block_t \*).
Functions used for (block_t \*) are also used for (m_block_t \*) by casting pointers.

All non-mini (blocks of size 16) free blocks are footerless. To find the
previous block in function coalesce_block, the second to last bit of each
block is set if the previous block is allocated. To know if the previous
block is a mini, the third to last bit of each block is set if so.

```c
/**
 * @file mm.c
 * @brief A 64-bit struct-based explicit seg list memory allocator
 *
 * 15-213: Introduction to Computer Systems
 *
 * @author holly liu <hsliu@andrew.cmu.edu>
 */

#include <assert.h>
#include <inttypes.h>
#include <stdbool.h>
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#include "memlib.h"
#include "mm.h"

// ... omitted lines ... //

/** @brief Stores the block header. If allocated, stores the footerless payload.
 *         If free, stores the previous and next pointers to other blocks.
 *         Block size is always > 16.
 *
 * The block is used as memory that stores a payload of information to
 * simulate a memory allocator.
 */
struct block
{
    /** @brief Header contains size + allocation flag */
    word_t header;

    /**
     * @brief A pointer to the block payload.
     *
     * WARNING: DO NOT cast this pointer to/from other types! Instead, you
     * should use a union to alias this zero-length array with another struct,
     * in order to store additional types of data in the payload memory.
     */
    union
    {
        struct
        {
            block_t *prev;
            block_t *next;
        };
        char payload[0];
    };
};

struct m_block
{
    /** @brief Header contains size + allocation flag */
    word_t header;

    /**
     * @brief A pointer to the block payload.
     *
     * WARNING: DO NOT cast this pointer to/from other types! Instead, you
     * should use a union to alias this zero-length array with another struct,
     * in order to store additional types of data in the payload memory.
     */
    union
    {
        struct
        {
            m_block_t *next;
        };
        char payload[0];
    };
};

/* Global variables */

/** @brief Pointer to first block in the heap. Used for checkheap. */
static block_t *heap_start = NULL;
/** @brief Seglist buckets of blocks of size > 16 */
static block_t *seg_list[14];
static const size_t c = min_block_size;
static const size_t seg_size[14] = {c * 2, c * 3, c * 5, c * 9, c * 12, c * 16, c * 38,
                                    c * 65, c * 129, c * 257, c * 513, c * 1025, c * 2049, c * 4097};
static const size_t list_num = 14;

/** @brief Mini seglist bucket. Only for block sizes == 16  */
static m_block_t *m_seg_list;
static const size_t min_seg_size = c;

// ... omitted lines ... //

/**
 * @brief Merges two free blocks if they are next to each other in the heap.
 *
 * Only uses "find_prev" if the previous block is a non-mini free block. Checks
 * for this by finding the p_alloc and p_mini bits of the input block. If the
 * previous block is a mini, then subtracts the minimum block size from the
 * block.
 *
 * @param[in] block
 * @return block
 * @post There are no free blocks next to the block
 */
static block_t *coalesce_block(block_t *block)
{

    // find_prev(block) does not work if previous footer is prologue
    block_t *p_block, *n_block = NULL;

    // find prev block and its alloc and mini status
    bool p_alloc = get_p_alloc(block);
    bool p_mini = get_p_m(block);

    // non-mini free blocks have footers, can use find_prev
    if (!p_alloc)
    {
        if (!p_mini)
        {
            p_block = find_prev(block);
        }
        else
        {
            // all minis are size 16
            p_block = (block_t *)((char *)block - min_seg_size);
        }
    }

    // find next block and its alloc
    n_block = find_next(block);
    bool n_alloc = get_alloc(n_block);

    // previous block packed as allocated by default
    if (p_alloc && n_alloc)
    {
        size_t i = find_index(block);
        insert_block(i, block);
    }
    else if (p_alloc && !n_alloc)
    {
        size_t i = find_index(n_block);
        remove_block(i, n_block);

        merge_blocks(block, n_block);

        size_t j = find_index(block);
        insert_block(j, block);
    }
    else if (!p_alloc && n_alloc)
    {
        // remove from seglist b/c size will change
        size_t i = find_index(p_block);
        remove_block(i, p_block);

        // p_block is never NULL- prologue is always allocated
        merge_blocks(p_block, block);

        size_t j = find_index(p_block);
        insert_block(j, p_block);

        block = p_block;
    }
    else
    { // !p_alloc && !n_alloc
        // remove b/c size will change
        size_t a = find_index(p_block);
        remove_block(a, p_block);

        size_t i = find_index(n_block);
        remove_block(i, n_block);

        merge_blocks(block, n_block);
        // p_block is never NULL- prologue is always allocated
        merge_blocks(p_block, block);

        size_t j = find_index(p_block);
        insert_block(j, p_block);

        block = p_block;
    }

    return block;
}

/**
 * @brief Allocates a free block.
 *
 * Searches seglists for a free block of size greater than 'size'. If no block
 * is found, extends the heap. Rewrites the allocated block. Splits block if
 * the unused portion of the block is greater than the minimum block size.
 *
 * @param[in] size
 * @return A pointer to the payload of the allocated block
 */
void *malloc(size_t size)
{
    dbg_requires(mm_checkheap(__LINE__));

    size_t asize;      // Adjusted block size
    size_t extendsize; // Amount to extend heap if no fit is found
    block_t *block;
    void *bp = NULL;

    // Initialize heap if it isn't initialized
    if (heap_start == NULL)
    {
        mm_init();
    }

    // Ignore spurious request
    if (size == 0)
    {
        dbg_ensures(mm_checkheap(__LINE__));
        return bp;
    }

    // Adjust block size to include overhead and to meet alignment requirements
    asize = round_up(size + wsize, dsize);

    // Search the free list for a fit
    block = find_fit(asize);

    // If no fit is found, request more memory, and then and place the block
    if (block == NULL)
    {
        // Always request at least chunksize
        extendsize = max(asize, chunksize);
        // block is in seglist b/c extend_heap updates list
        // check if last block in heap is allocated by getting bits from epi
        char *last_block = mem_heap_hi() - 7;
        bool p_alloc = get_p_alloc((block_t *)last_block);
        bool l_mini = get_p_m((block_t *)last_block);
        block = extend_heap(extendsize, p_alloc, l_mini);
        // extend_heap returns an error
        if (block == NULL)
        {
            return bp;
        }
    }

    // The block should be marked as free
    dbg_assert(!get_alloc(block));

    // block no longer free
    size_t i = find_index(block);
    remove_block(i, block);

    // Mark block as allocated
    size_t block_size = get_size(block);
    // free block is assumed to not be next to other free blocks
    write_block(block, block_size, true, true, get_p_m(block));

    // Try to split the block if too large
    // split_block updates explicit list and p_alloc bit of next block
    split_block(block, asize);

    bp = header_to_payload(block);

    dbg_ensures(mm_checkheap(__LINE__));
    return bp;
}

/**
 * @brief Frees an allocated block.
 *
 * Inserts the free block into seglist of size that is an upper bound of the
 * block size. Coalesces if there are free blocks next to the block. Sets its
 * allocation and mini status bits into the next block.
 *
 * @param[in] bp Payload of an allocated block
 * @param[out] A free block
 */
void free(void *bp)
{
    dbg_requires(mm_checkheap(__LINE__));

    if (bp == NULL)
    {
        return;
    }

    bool p_mini_next = false;

    block_t *block = payload_to_header(bp);
    size_t size = get_size(block);

    // The block should be marked as allocated
    dbg_assert(get_alloc(block));

    // Mark the block as free
    write_block(block, size, false, get_p_alloc(block), get_p_m(block));

    // Try to coalesce the block with its neighbors
    // coalesce will insert regardless if coalesced
    block = coalesce_block(block);

    // set p_alloc in next block to false, set mini bit in next block
    block_t *block_c_next = find_next(block);
    if (get_size(block) == min_seg_size)
    {
        p_mini_next = true;
    }
    block_c_next->header = pack(get_size(block_c_next), true, false,
                                p_mini_next);

    dbg_ensures(mm_checkheap(__LINE__));
}

```

For this project, I maxed out the throughput and utilization metrics but interestingly enough, could not figure out the final 1% to achieve a grade of 100%.
