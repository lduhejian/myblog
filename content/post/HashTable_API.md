---
title: "Php7 HashTable API 注释"
date: 2020-07-17
draft: true
tags: [php7, php, HashTable, API]
description: "HashTable API 注释"
---
本文中的代码来自 php-7.1.0

```
ZEND_API void ZEND_FASTCALL _zend_hash_init(HashTable *ht, uint32_t nSize, dtor_func_t pDestructor, zend_bool persistent ZEND_FILE_LINE_DC)
{
	GC_REFCOUNT(ht) = 1; // 初始化 refcount = 1
	GC_TYPE_INFO(ht) = IS_ARRAY; // 初始化 type_info 为 IS_ARRAY = 7
	ht->u.flags = (persistent ? HASH_FLAG_PERSISTENT : 0) | HASH_FLAG_APPLY_PROTECTION | HASH_FLAG_STATIC_KEYS; // 设置 HashTable 的标记，TODO 三个宏定义的具体含义
	ht->nTableSize = zend_hash_check_size(nSize);
	ht->nTableMask = HT_MIN_MASK; // 初始化 nTableMask = -2
	HT_SET_DATA_ADDR(ht, &uninitialized_bucket); // 初始化 arData
	ht->nNumUsed = 0;
	ht->nNumOfElements = 0;
	ht->nInternalPointer = HT_INVALID_IDX; // TODO HT_INVALID_IDX 是什么意思？
	ht->nNextFreeElement = 0;
	ht->pDestructor = pDestructor; // pDestructor 是函数指针
}
```

```
static zend_always_inline uint32_t zend_hash_check_size(uint32_t nSize)
{
#if defined(ZEND_WIN32)
	unsigned long index;
#endif

	/* Use big enough power of 2 */
	/* size should be between HT_MIN_SIZE and HT_MAX_SIZE */
	if (nSize < HT_MIN_SIZE) {
		nSize = HT_MIN_SIZE;
	} else if (UNEXPECTED(nSize >= HT_MAX_SIZE)) {
		zend_error_noreturn(E_ERROR, "Possible integer overflow in memory allocation (%u * %zu + %zu)", nSize, sizeof(Bucket), sizeof(Bucket));
	}

#if defined(ZEND_WIN32)
	if (BitScanReverse(&index, nSize - 1)) {
		return 0x2 << ((31 - index) ^ 0x1f);
	} else {
		/* nSize is ensured to be in the valid range, fall back to it
		   rather than using an undefined bis scan result. */
		return nSize;
	}
#elif (defined(__GNUC__) || __has_builtin(__builtin_clz))  && defined(PHP_HAVE_BUILTIN_CLZ)
	return 0x2 << (__builtin_clz(nSize - 1) ^ 0x1f); //TODO p 0x2 << (__builtin_clz(nSize - 1) ^ 0x1f) No symbol "__builtin_clz" in current context.
#else
	nSize -= 1;
	nSize |= (nSize >> 1);
	nSize |= (nSize >> 2);
	nSize |= (nSize >> 4);
	nSize |= (nSize >> 8);
	nSize |= (nSize >> 16);
	return nSize + 1;
#endif
}
```
