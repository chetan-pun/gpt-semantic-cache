
# GPT Semantic Cache: Reducing LLM Costs and Latency via Semantic Embedding Caching

This repository is the official implementation of [GPT Semantic Cache: Reducing LLM Costs and Latency via Semantic Embedding Caching](https://arxiv.org/pdf/2411.05276). 

The GPT Semantic Cache is a Node.js package that provides a semantic caching mechanism for GPT responses. By leveraging semantic embeddings and approximate nearest neighbors search, the package efficiently caches and retrieves GPT responses based on the semantic similarity of user queries. This reduces redundant API calls to GPT models, saving time and costs, and improving response times for end-users. Queries with similar meaning are retrieved from cache saving the cost associated with an API.
## Installation

```bash
npm install gpt-semantic-cache
```

## Quick Start

Here's a quick example to get you started:

```javascript
const { SemanticGPTCache } = require('gpt-semantic-cache');

(async () => {
  const cache = new SemanticGPTCache({
    embeddingOptions: {
      type: 'openai',
      openAIApiKey: 'YOUR_OPENAI_API_KEY',
    },
    gptOptions: {
      openAIApiKey: 'YOUR_OPENAI_API_KEY',
      model: 'gpt-3.5-turbo',
    },
    cacheOptions: {
      redisUrl: 'redis://localhost:6379',
      similarityThreshold: 0.8,
      cacheTTL: 3600, // Cache Time-To-Live in seconds
      embeddingSize: 1536, // OpenAI's embedding size
    },
  });

  await cache.initialize();

  const response = await cache.query('What is the capital of France?');
  console.log(response);
})();
```

## Usage

### Initialization

To initialize the GPT semantic cache, you need to provide configuration options for embeddings, GPT model, and caching.

```javascript
const cache = new SemanticGPTCache({
  embeddingOptions: {
    type: 'local', // 'openai' or 'local'
    modelName: 'sentence-transformers/all-MiniLM-L6-v2', // Only for local models
    openAIApiKey: 'YOUR_OPENAI_API_KEY', // Only for OpenAI embeddings
  },
  gptOptions: {
    openAIApiKey: 'YOUR_OPENAI_API_KEY',
    model: 'gpt-3.5-turbo', // GPT model to use to query gpt if cache misses
    promptPrefix: 'You are an AI assistant.',
  },
  cacheOptions: {
    redisUrl: 'redis://localhost:6379',
    similarityThreshold: 0.8, // Cosine similarity threshold for cache hits
    cacheTTL: 3600, // Time-to-live for cache entries in seconds
    embeddingSize: 384, // Embedding size (384 for local models, 1536 for OpenAI)
  },
});

await cache.initialize();
```

**Initialization Options Explained:**

- **embeddingOptions**:
  - `type`: `'openai'` or `'local'`. Specifies the source of embeddings.
  - `modelName`: The name of the local embedding model to use (e.g., `'sentence-transformers/all-MiniLM-L6-v2'`).
  - `openAIApiKey`: Your OpenAI API key (required if `type` is `'openai'`).

- **gptOptions**:
  - `openAIApiKey`: Your OpenAI API key for accessing the GPT model.
  - `model`: The GPT model to use (e.g., `'gpt-3.5-turbo'`) in case of cache miss.
  - `promptPrefix`: An optional string to prepend to every prompt sent to the GPT model.

- **cacheOptions**:
  - `redisUrl`: The URL of your Redis instance (e.g., `'redis://localhost:6379'`).
  - `similarityThreshold`: A number between 0 and 1 representing the cosine similarity threshold for cache hits.
  - `cacheTTL`: The time-to-live for cache entries in seconds.
  - `embeddingSize`: The dimensionality of the embeddings used (e.g., `384` for local models, `1536` for OpenAI).

### Querying

To query the cache and get a response:

```javascript
const response = await cache.query('Your query here', 'Additional context if any');
console.log(response);
```

- If a similar query exists in the cache (based on the similarity threshold), the cached response is returned.
- If no similar query is found, the GPT API is called, and the response is cached for future queries.


## Evaluation

To evaluate use:
```javascript
import { SemanticGPTCache } from './index';
import data from './test_dataset/customer_qa.json' with {type: "json"};
import prev from './test_dataset/similar_customer.json' with {type: "json"};

async function main() {
  const cache = new SemanticGPTCache({
    embeddingOptions: {
      type: 'local',
      modelName: 'Xenova/all-MiniLM-L6-v2',
      openAIApiKey: process.env.OPENAI_API_KEY || '',
    },
    gptOptions: {
      openAIApiKey: process.env.OPENAI_API_KEY || '',
      model: 'gpt-4o-mini-2024-07-18',
      promptPrefix: 'You are a helpful assistant and a technical support assistant for a 3D printer, you will limit your result to 5 sentences',
    },
    cacheOptions: {
      redisUrl: process.env.REDIS_URL,
      similarityThreshold: 0.8, 
      cacheTTL: 86400,
    },
  });
  await cache.initialize();
  await cache.clearCache();
  for (const item of prev.questions) {
    const response = await cache.query(item.question,context)
  }
  console.log("Api hit : " + cache.getApiHit())
  console.log("Cache hit : "+cache.getCacheHit())
  console.log("Positive Hit : " + cache.getPositiveHit())
  console.log("Negative hit :" + cache.getNegativeHit())
  // Optionally clear the cache at the end
   await cache.clearCache();
}
main(); 
```

The code and the dataset we have used is available in [Evalute gpt-semantic-cache](https://anonymous.4open.science/r/gpt-semantic-cache-test-56CE/test.ts). 


To verify the appropriateness of cached responses, we employed GPT-4o Mini to evaluate whether the retrieved cached response was valid for the test query. For each cache hit, both the test query and the original cached question were provided to the model, which returned a binary verdict indicating whether the queries were semantically similar
and whether the cached response was accurate. This validation step enabled us to quantify the accuracy of the caching mechanism. Additionally, the response times for queries were measured and averaged for both the caching system and the traditional method (without cache). This allowed us to determine the performance improvement in terms of query response time.


## Results

### Cache Hit per 500 queries in each category and the number of positive hits

| Category                               | Cache Hit | Positive Hits |
|----------------------------------------|-----------|---------------|
| Basics of Python Programming           | 335       | 310           |
| Technical Support Related to Network   | 335       | 326           |
| Questions Related to Order and Shipping| 344       | 331           |
| Customer Shopping QA                   | 308       | 298           |


**GPT Semantic Cache demonstrate a significant reduction in API calls and improvement in response times across all evaluated query categories. The cache hit rates of up to 68.8% and positive hit rates of the cache hit rates exceeding 97% highlight the system’s effectiveness in identifying and leveraging semantic similarity for query handling. However, it is essential to contextualize these outcomes within both their strengths and potential limitations.**


## Contributing
We welcome issues and pull requests!

- Fork and create a feature branch  
- Add tests where applicable  
- Open a PR describing your change and rationale  

This project is licensed under the MIT License.
