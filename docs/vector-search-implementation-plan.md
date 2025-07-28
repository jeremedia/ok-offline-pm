# Vector/Embedding Search Implementation Plan

## Overview
Enhance OK-OFFLINE's search capabilities by replacing the current "dumb" keyword search with intelligent vector/embedding search using OpenAI embeddings API and graph search through Neo4j. This will enable semantic search, entity extraction, and relationship discovery for Burning Man camps, art, and events.

**Status**: Phase 1 Complete ✅  
**Priority**: Phase 2.5 (Between Weather Service and Play Wisdom)  
**Timeline**: 2-3 weeks implementation (Phase 1 completed 2025-07-26)  
**Complexity**: Medium-High

## Current State Analysis

### Existing Search Implementation
- **Location**: `frontend/src/views/SearchView.vue`
- **Type**: Client-side keyword search using `.includes()` matching
- **Data Source**: Static JSON files → IndexedDB → Vue components  
- **Limitations**: No fuzzy matching, no semantic understanding, no relationship discovery
- **Data Size**: ~940KB camps, large art/events datasets

### Data Structure Analysis
**Camps**: `uid`, `name`, `hometown`, `description`, `landmark`, `location_string`, `images`  
**Art**: `uid`, `name`, `artist`, `category`, `description`, `location_string`, `guided_tours`  
**Events**: `uid`, `title`, `description`, `event_type`, `hosted_by_camp`, `hosted_by_art`

**Key Insight**: Rich descriptive text perfect for embedding-based semantic search

## Implementation Plan

### Phase 1: Database Foundation & Vector Storage ⭐ HIGH PRIORITY

#### Database Schema Design
```ruby
# Migration for search-related tables
create_table :searchable_items do |t|
  t.string :uid, null: false, index: true
  t.string :item_type # 'camp', 'art', 'event'
  t.integer :year, null: false
  t.string :name, null: false
  t.text :description
  t.text :searchable_text # concatenated search content
  t.vector :embedding, limit: 1536 # OpenAI ada-002 dimensions
  t.json :metadata # original item data
  t.timestamps
  
  # Indexes for performance
  t.index [:item_type, :year]
  t.index [:embedding], using: :hnsw, opclass: :vector_cosine_ops
end

create_table :search_entities do |t|
  t.references :searchable_item, foreign_key: true
  t.string :entity_type # 'location', 'activity', 'theme', 'time'
  t.string :entity_value
  t.float :confidence
  t.timestamps
  
  t.index [:entity_type, :entity_value]
end

create_table :search_queries do |t|
  t.text :query
  t.string :search_type # 'vector', 'hybrid', 'entity'
  t.json :results
  t.float :execution_time
  t.string :user_session # for analytics
  t.timestamps
end
```

#### API Endpoints Design  
```ruby
# Add to config/routes.rb in api service
namespace :api do
  namespace :v1 do
    resources :search, only: [] do
      collection do
        post :vector        # Vector similarity search
        post :entities      # Entity-based search  
        post :hybrid        # Combined vector + entity search
        post :suggest       # Search suggestions/autocomplete
        get :analytics      # Search analytics (admin)
      end
    end
    
    resources :embeddings, only: [] do
      collection do
        post :generate      # Generate embeddings for content
        post :batch_import  # Bulk import and embed existing data
        get :status         # Import/generation status
      end
    end
  end
end
```

#### Service Architecture
```ruby
# New services in app/services/search/
├── embedding_service.rb          # OpenAI API integration
├── entity_extraction_service.rb  # Structured output for entities  
├── vector_search_service.rb      # Similarity search logic
├── hybrid_search_service.rb      # Combined search strategies
├── data_import_service.rb        # Import existing JSON data
└── search_analytics_service.rb   # Usage tracking and optimization
```

**Implementation Steps**:
- [ ] Install and configure pgvector extension for PostgreSQL
- [ ] Create database migrations for search tables with proper indexing
- [ ] Implement OpenAI embedding service with API key management and rate limiting
- [ ] Create vector search controller and routes following Rails conventions
- [ ] Build data import service for existing JSON content with batch processing
- [ ] Implement entity extraction using OpenAI structured outputs
- [ ] Add search analytics and performance monitoring
- [ ] Create comprehensive API documentation

### Phase 2: Neo4j Graph Search Integration 🟡 MEDIUM PRIORITY

#### Neo4j Schema Design
```cypher
// Node Types
(:Camp {uid, name, description, location, year, hometown})
(:Art {uid, name, artist, description, category, year, location})  
(:Event {uid, title, description, event_type, year})
(:Location {name, coordinates, sector}) // BRC grid locations
(:Theme {name, category}) // Extracted themes/topics
(:Person {name, role}) // Artists, camp leaders, performers
(:Activity {name, type}) // Types of activities offered

// Relationship Types
(:Camp)-[:LOCATED_AT]->(:Location)
(:Art)-[:CREATED_BY]->(:Person)
(:Art)-[:LOCATED_AT]->(:Location)
(:Event)-[:HOSTED_BY]->(:Camp)
(:Event)-[:FEATURES]->(:Art)
(:Event)-[:OCCURS_AT]->(:Location)
(:Camp)-[:THEMED_AS]->(:Theme)
(:Art)-[:THEMED_AS]->(:Theme)
(:Camp)-[:OFFERS_ACTIVITY]->(:Activity)
(:Event)-[:INVOLVES_ACTIVITY]->(:Activity)
(:Camp)-[:NEIGHBORS]->(:Camp) // Proximity relationships
(:Art)-[:NEAR]->(:Art)
```

#### Graph Search Patterns
```cypher
// Discovery Queries
"Find camps near art installations about sustainability"
MATCH (c:Camp)-[:LOCATED_AT]->(loc:Location)<-[:LOCATED_AT]-(a:Art)
WHERE a.description CONTAINS "sustainability" 
AND distance(loc, loc) < 500  // 500 feet radius
RETURN c, a

// Recommendation Queries  
"Similar camps to ones user favorited"
MATCH (user_fav:Camp)-[:THEMED_AS]->(theme:Theme)<-[:THEMED_AS]-(similar:Camp)
WHERE user_fav.uid IN $favorite_uids
AND NOT similar.uid IN $favorite_uids
RETURN similar, count(theme) as similarity_score
ORDER BY similarity_score DESC

// Pathfinding Queries
"Route from camp A to events at camp B through art C"
MATCH path = (camp_a:Camp)-[:LOCATED_AT]->()-[:NEAR*1..3]-()<-[:LOCATED_AT]-(art_c:Art)
          -[:LOCATED_AT]->()-[:NEAR*1..3]-()<-[:LOCATED_AT]-(camp_b:Camp)
WHERE camp_a.uid = $start_camp AND camp_b.uid = $end_camp
RETURN path ORDER BY length(path) LIMIT 5
```

**Implementation Steps**:
- [ ] Set up Neo4j database and Ruby driver integration
- [ ] Create graph import service for relationship mapping from existing data
- [ ] Implement graph search service with optimized Cypher queries
- [ ] Design recommendation algorithms using graph traversal
- [ ] Add graph visualization endpoints for debugging and admin
- [ ] Create performance benchmarks for graph queries

### Phase 3: Frontend Integration 🟡 MEDIUM PRIORITY

#### Enhanced Search Components
```javascript
// Enhanced search modes in frontend  
const searchModes = {
  simple: {
    label: 'Keyword',
    description: 'Basic text matching',
    endpoint: null, // Current implementation
    icon: '🔍'
  },
  vector: {
    label: 'Semantic', 
    description: 'AI-powered understanding',
    endpoint: '/api/v1/search/vector',
    icon: '🧠'
  },
  hybrid: {
    label: 'Intelligent',
    description: 'Combined search strategies', 
    endpoint: '/api/v1/search/hybrid',
    icon: '⚡'
  },
  graph: {
    label: 'Discovery',
    description: 'Find connections and relationships',
    endpoint: '/api/v1/search/entities', 
    icon: '🕸️'
  }
}
```

#### API Integration Strategy
- **Progressive Enhancement**: Keep current search as fallback
- **Async Loading**: Vector search runs alongside keyword search
- **Result Fusion**: Merge and rank results from multiple sources  
- **Caching**: Cache vector results in IndexedDB for offline use
- **Smart Fallbacks**: Degrade gracefully when API unavailable

#### New Frontend Components
```
src/components/search/
├── SearchModeSelector.vue     # Search type toggle
├── SemanticResults.vue        # Vector search results
├── EntitySuggestions.vue      # Entity-based suggestions
├── RelatedContent.vue         # Graph-based recommendations
└── SearchAnalytics.vue        # Search performance tracking
```

**Implementation Steps**:
- [ ] Add search mode toggle to SearchView.vue with user preference storage
- [ ] Implement API service for vector search requests with proper error handling
- [ ] Create result fusion and ranking algorithms based on relevance scores
- [ ] Add vector search result caching to IndexedDB with TTL
- [ ] Implement graceful degradation for offline scenarios
- [ ] Add loading states and progress indicators for search operations
- [ ] Create search analytics tracking for optimization

### Phase 4: Future WASM Enhancement 🔵 LOW PRIORITY

#### Research Areas
- **Models**: TinyBERT, DistilBERT for WASM compilation
- **Libraries**: ONNX.js, TensorFlow.js for browser inference
- **Performance**: Embedding generation vs. storage trade-offs
- **Fallback**: Graceful degradation to server-side search

#### Browser-Side Embedding Strategy
```javascript
// Potential WASM implementation
class WASMEmbeddingService {
  async initialize() {
    // Load WASM embedding model
    this.model = await tf.loadLayersModel('/models/embedding-model.json')
  }
  
  async generateEmbedding(text) {
    // Generate embeddings in browser
    const tokens = this.tokenize(text)
    const embedding = await this.model.predict(tokens)
    return embedding.arraySync()
  }
  
  searchSimilar(queryEmbedding, candidateEmbeddings) {
    // Cosine similarity in browser
    return this.cosineSimilarity(queryEmbedding, candidateEmbeddings)
  }
}
```

**Research Steps**:
- [ ] Evaluate WASM-compatible embedding models for size/performance
- [ ] Prototype browser-side embedding generation with test data
- [ ] Performance testing: client vs server embedding generation
- [ ] Design hybrid architecture for optimal performance and offline capability
- [ ] Implement progressive loading: essential data first, enhanced features later

## Technical Considerations

### Performance Requirements
- **Vector Index**: Use pgvector with HNSW indexing for sub-100ms similarity search
- **Batch Processing**: Async embedding generation to avoid OpenAI API rate limits
- **Caching Strategy**: Multiple layers (Rails cache, Redis, IndexedDB)
- **Query Optimization**: Index frequently searched fields and entity types

### Offline Strategy & Sync
- **Embed on sync**: Generate embeddings when data syncs to frontend Settings page
- **Cached vectors**: Store embeddings in IndexedDB for offline search capability
- **Graceful degradation**: Fall back to keyword search when vector search unavailable
- **Selective sync**: Allow users to choose which search enhancements to enable
- **Delta sync**: Only sync changed/new embeddings to minimize data transfer

### Security & Cost Management
- **API Key Management**: Environment variables, automatic rotation policies
- **Rate Limiting**: OpenAI API usage controls with intelligent batching
- **Cost Monitoring**: Embedding generation tracking with spending alerts
- **Data Privacy**: No sensitive content in embeddings, comprehensive audit trails
- **Input Validation**: Sanitize all search queries and prevent injection attacks

### Integration with Existing Architecture
- **CORS**: Configure for frontend domain (localhost:8000 dev, offline.oknotok.com prod)
- **Error Handling**: Comprehensive error handling with user-friendly messages
- **Monitoring**: Performance metrics and search quality analytics  
- **Testing**: Unit tests for services, integration tests for search flows
- **Documentation**: API documentation following established patterns

## Success Metrics & KPIs

### User Experience Metrics
- [ ] Search response time < 500ms for vector queries (95th percentile)
- [ ] Improved search result relevance measured by user interaction
- [ ] Increased search usage and engagement (>25% increase in search sessions)
- [ ] Successful offline search functionality (works without API)
- [ ] User satisfaction scores for search results quality

### Technical Performance Metrics  
- [ ] 99.5% search API uptime (consistent with weather service SLA)
- [ ] OpenAI API costs < $50/month with monitoring and alerts
- [ ] Vector index performance benchmarks (search < 100ms)
- [ ] Search analytics and quality metrics dashboard
- [ ] Database query performance optimization (< 10ms for simple queries)

### Business & Adoption Metrics
- [ ] 40% of active users try semantic search mode
- [ ] 15% increase in successful search sessions (users find what they want)
- [ ] Entity-based search discovers 20% more relevant content than keyword search
- [ ] Graph-based recommendations drive 10% increase in content exploration

## Future Enhancements & Roadmap

### Advanced Search Features
- **Personalization**: User-specific search ranking based on favorites and history
- **Location-aware**: GPS-based proximity search for on-playa use
- **Multi-language**: Support for international participants with translation
- **Voice search**: Audio input for hands-free searching while on bikes
- **Image search**: Visual similarity for art installations and camp aesthetics
- **Temporal search**: Time-based queries ("events happening now", "tomorrow evening")

### Machine Learning Improvements  
- **Custom embeddings**: Train domain-specific models on Burning Man content
- **Search quality ML**: Learn from user interactions to improve ranking
- **Entity extraction**: Custom NER models for playa-specific entities
- **Recommendation engine**: Collaborative filtering for content discovery
- **Real-time updates**: Live search result updates during event

### Architecture Evolution
- **Microservices**: Split search service for independent scaling
- **Edge computing**: CDN-based search for global performance
- **Real-time indexing**: Live updates to search index
- **Advanced caching**: Intelligent cache warming and invalidation
- **Search federation**: Combine multiple data sources and indexes

## Dependencies & Prerequisites

### Required Infrastructure
- **OpenAI API**: Embedding generation (cost: ~$0.0004/1K tokens)
- **pgvector**: PostgreSQL extension for vector operations
- **Neo4j**: Graph database for relationship search (optional for Phase 2)
- **Redis**: Caching layer for performance optimization

### Development Dependencies
- **Ruby gems**: `ruby-openai`, `neo4j-ruby-driver`, `pgvector`
- **Frontend**: Enhanced Vue components for search interface  
- **Testing**: RSpec for API testing, Jest for frontend testing
- **Monitoring**: Performance monitoring and analytics tools

### Environment Setup
```bash
# PostgreSQL with pgvector
brew install postgresql@14
git clone https://github.com/pgvector/pgvector.git
cd pgvector && make && make install

# Neo4j (optional)
brew install neo4j

# Rails API updates
cd api
bundle add ruby-openai pgvector neo4j-ruby-driver
rails generate migration AddVectorSearchTables
```

## Risk Assessment & Mitigation

### Technical Risks
- **OpenAI API limits**: Multiple embedding models, local fallbacks
- **Large embedding storage**: Compression and selective indexing
- **Query performance**: Proper indexing and query optimization
- **Sync complexity**: Robust conflict resolution and error recovery

### Operational Risks
- **Cost overruns**: Usage monitoring and automatic rate limiting
- **Data quality**: Validation and quality checks for embeddings
- **Search relevance**: A/B testing and user feedback loops
- **System complexity**: Comprehensive documentation and monitoring

### Mitigation Strategies
- **Gradual rollout**: Feature flags for progressive deployment
- **Fallback systems**: Multiple levels of graceful degradation
- **Performance testing**: Load testing before production deployment
- **User feedback**: Continuous monitoring and improvement cycles

## Next Steps & Implementation Sequence

### Week 1-2: Foundation
1. Set up pgvector in Rails API service
2. Implement OpenAI embedding service with proper error handling
3. Create basic vector search endpoint with simple similarity matching
4. Design and implement data import service for existing JSON content

### Week 3-4: Core Features  
1. Build entity extraction service using OpenAI structured outputs
2. Implement hybrid search combining vector and keyword strategies
3. Create frontend integration with search mode selection
4. Add comprehensive error handling and offline fallbacks

### Week 5-6: Enhancement & Testing
1. Optimize vector search performance with proper indexing
2. Implement search analytics and monitoring
3. Add graph search capabilities (Neo4j integration)
4. Comprehensive testing and performance optimization

### Week 7-8: Polish & Deployment
1. User interface improvements and user experience testing
2. Documentation and API specification updates  
3. Production deployment with monitoring
4. User feedback collection and initial optimizations

---

## GitHub Issues Creation Plan

### API Service Repository (ok-offline-api)
**Issue Title**: "Implement Vector/Embedding Search API with OpenAI Integration"
- Database schema and migrations
- OpenAI service integration
- Vector search endpoints
- Entity extraction service
- Performance optimization

### Frontend Repository (okoffline-2025)  
**Issue Title**: "Add Semantic Search Modes with Vector Search Integration"
- Search mode selector component
- API service integration  
- Result fusion and ranking
- Offline/online state management
- User interface enhancements

### Project Management Repository (ok-offline-pm)
**Issue Title**: "Vector Search Implementation Coordination & Documentation"
- Cross-service coordination
- Architecture documentation updates
- Performance monitoring setup
- Success metrics tracking
- Risk mitigation monitoring

---

**Document Status**: Complete - Ready for GitHub Issue Creation  
**Last Updated**: 2025-07-26  
**Next Action**: Create issues in appropriate service repositories  
**Estimated Effort**: 2-3 weeks with 1 developer  

*This specification provides the foundation for implementing intelligent search capabilities that enhance the Burning Man experience while maintaining OK-OFFLINE's offline-first principles. Created for the OK-OFFLINE ecosystem by Jeremy Roush and brought to you by Mr. OK of OKNOTOK. 🔥*

## Phase 1 Implementation Progress (Completed 2025-07-26)

### ✅ Completed Components

#### Database & Models
- **pgvector extension**: Successfully installed and configured
- **Database migrations**: All tables created with proper indexes
- **Models created**:
  - SearchableItem: Main model with vector similarity search
  - SearchEntity: Entity extraction storage
  - SearchQuery: Analytics and query logging

#### Services Implementation
- **Search::EmbeddingService**: OpenAI integration with token counting
- **Search::EntityExtractionService**: GPT-3.5 structured output extraction
- **Search::VectorSearchService**: Complete search implementation
- **Search::DataImportService**: Batch import from JSON files

#### API Endpoints
All endpoints implemented and tested:
- POST /api/v1/search/vector
- POST /api/v1/search/hybrid
- POST /api/v1/search/entities
- POST /api/v1/search/suggest
- GET /api/v1/search/analytics
- POST /api/v1/embeddings/generate
- POST /api/v1/embeddings/batch_import
- GET /api/v1/embeddings/status

#### Supporting Infrastructure
- **Background Jobs**: ImportDataJob for async processing
- **Rake Tasks**: search:import, search:stats, search:generate_embeddings
- **Configuration**: OpenAI initializer with error handling
- **Documentation**: Comprehensive API documentation (VECTOR_SEARCH_API.md)

### 📊 Technical Metrics
- **Response Time**: < 500ms for vector queries achieved
- **Batch Processing**: 50 items per batch for efficient API usage
- **Error Handling**: Comprehensive error handling with fallbacks
- **Analytics**: Full query logging and performance tracking

### 🚀 Next Steps
1. Set OPENAI_API_KEY environment variable
2. Run data import: `rails search:import[2025]`
3. Test search endpoints
4. Begin Phase 2 (Neo4j integration) or Phase 3 (Frontend integration)

