### The Architecture: It's a VST "Skeleton"
There are two key technical concepts that make this possible:

Separation of Processing and UI (The VST3 Way): The VST3 format is designed to separate its "brain" (audio processor) from its "face" (edit controller). This separation was specifically created so a host could run each component in a different context, even on different computers . Your VST file would contain the "face" and the networking code, while the server would run the "brain" .

External Runtime (The "Server"): The server, running on any OS, would host the actual model's runtime. This is a proven method. Projects like AudioGridder already allow DAWs to offload VST processing to another machine . The Fourier Audio transform.engine is a commercial product doing this for live sound, running VST3 plugins on a dedicated server rack unit .

### 💡 How This Could Work for Your Free/Donation Model
You can definitely build this on a free or donation basis. The user would run a simple server application you provide, which then hosts the LLM model. This server could connect to a shared community runtime or just be a standalone app.

### 🛠️ Example of "Skeleton" VSTs that Connect Out
There are already working examples where a VST is essentially a "client" for a remote service:

OBSIDIAN-Neural: This VST3 plugin previously relied on a server connection for its AI generation. A major update introduced a fully offline, local model to eliminate server dependencies, demonstrating a clear shift from the server-based model to a local one to solve its main pain point: "server dependencies and resource requirements" .

Google's The Infinite Crate: This VST prototype integrates a generative music model. The developers explicitly state that the core model "is not capable of running locally on consumer hardware," so the plugin requires an API key and internet access to function .

tx2vst: This project uses an AI agent to generate the code for a VST plugin from a text prompt, rather than hosting a runtime model . It's an example of an LLM playing a central role in the VST ecosystem.

Audio Plugin MCP Server: This is a local server that gives AI agents (like Claude) the ability to load and interact with VST plugins programmatically . It represents the inverse idea—an external server controlling the plugin.

### 🔧 Potential Implementation Paths
Building this yourself might be simpler than you think, especially with emerging tools:

Use an "AI-First" Plugin Framework: The Audio Plugin Coder (APC) is an open-source framework designed to guide LLM agents through the entire plugin development process . This could help you generate the "skeleton" VST code.

Leverage Existing Remote Hosting Systems: You could adapt a tool like AudioGridder, which already creates a network bridge for audio and MIDI and supports VST3 plugins . Your plugin could connect to it.

Build from Scratch with JUCE: Use the JUCE framework, which underpins many projects, to build your plugin. Its architecture supports creating a plugin that connects to a remote service via a protocol like WebSockets .

### ⚠️ Considerations for Your Model
If you're thinking of a community or donation-based server, this is the hardest part. As mentioned in the context of OBSIDIAN-Neural, models that produce high-quality audio in real-time are extremely demanding on hardware . Running a free server for hundreds of users would be costly and technically challenging. For the initial version, it might be more practical to follow OBSIDIAN-Neural's local model path , or use a server for "assistance" tasks (like generating a preset) rather than for real-time audio generation.

To focus the possibilities further, could you share whether your goal is to host the LLM model on your own server, or distribute it for users to run locally? This is the key decision that will shape the architecture.


'''
Oakmont Copeland <oakmontcopeland@gmail.com>
Uni
Oakmont Copeland <oakmontcopeland@gmail.com> To: Oakmont Copeland <oakmontcopeland@gmail.com>
Dockerfile
# Dockerfile
FROM ruby:3.3-slim
RUN apt-get update && apt-get install -y build-essential && rm -rf /var/lib/apt/lists/*
WORKDIR /app
# Install gems
COPY Gemfile Gemfile.lock ./
RUN bundle install
# Copy app code (will be overmounted for dev)
COPY . .
# Ensure index directory exists
RUN mkdir -p /root/.knowos
# Default command: run the application in subtractive search mode
CMD ["ruby", "bin/knowos"]
Fri, Jul 17, 2026 at 5:32 AM
Hot reload watch docker-compose.yml
# docker-compose.yml
version: '3.8'
services:
knowos:
build: .
container_name: knowos_os
volumes:
- ./:/app # live code mount
- ~/Documents:/data # your real documents
- knowos_index:/root/.knowos
environment:
KNOWOS_WATCH_DIRS: "/data"
KNOWOS_TOP_K: 5
KNOWOS_VECTOR_DIM: 512
stdin_open: true
tty: true
# Optional hot-reload sidecar using watchexec (or nodemon-like)
watcher:
image: alpine:latest
command: >
sh -c "apk add --no-cache watchexec &&
watchexec -r -w /app -- docker-compose restart knowos"
volumes:
- ./:/app
- /var/run/docker.sock:/var/run/docker.sock
volumes:
knowos_index:
Phinodemap
class PhiNodeMap
GOLDEN_ANGLE = 2.0 * Math::PI * (2.0 - 1.618033988749895) # φ-1 * 2π
def initialize(dim:)
@dim = dim
@items = [] # sorted by phi coordinate
end
# Compute the phi coordinate for a normalized embedding vector
def phi_coordinate(vector)
# Project onto a 2D Fibonacci spiral and compute 1D coordinate
x = 0.0; y = 0.0
vector.each_with_index do |val, i|
theta = i * GOLDEN_ANGLE
radius = Math.sqrt(i.to_f / @dim) # equal area
x += val * radius * Math.cos(theta)
y += val * radius * Math.sin(theta)
end
# Map (x,y) to a radial angle and distance, then to a single natural number
angle = Math.atan2(y, x)
dist = Math.sqrt(x*x + y*y)
# Use a scaling factor to convert to integer index (golden ratio base)
(dist * 1000000 + angle * 1000).to_i
end
# Add an item
def add(item)
n = phi_coordinate(item.embedding)
@items << { n: n, item: item }
@items.sort_by! { |entry| entry[:n] } # keep sorted (could insert in order)
end
# Remove an item (optional)
def remove(item_id)
@items.reject! { |entry| entry[:item].id == item_id }
end
# Find items whose content contains substring, using phi proximity
def find_substring(substring, embedder, radius: 1000)
# Embed the substring to get its phi coordinate
sub_emb = embedder.encode([substring]).first
target_n = phi_coordinate(sub_emb)
# Binary search for the neighbourhood
left = @items.bsearch_index { |entry| entry[:n] >= target_n - radius } || 0
right = @items.bsearch_index { |entry| entry[:n] > target_n + radius } || @items.size
# Check actual substring presence in the narrow range
results = []
(left...right).each do |i|
item = @items[i][:item]
if item.content.downcase.include?(substring.downcase)
results << item
end
end
results
end
end
class Application
def initialize(...)
# ... existing setup ...
@phi_map = PhiNodeMap.new(dim: config.vector_dim)
end
# When indexing, also add to phi map
def index_file(path, lat: nil, lng: nil)
# ... existing versioned add ...
item = @vindex.latest_items[file_id] # after adding
@phi_map.add(item) if item
end
# In subtractive search loop
def run_subtractive_search
# ... initial scan, watcher ...
current_items = []
loop do
print "\n> "
input = gets&.chomp
break if input.nil? || input.downcase == 'exit'
if input.downcase.start_with?('find:')
find_text = input[5..].strip
# Use phi map to quickly locate items within current_items that contain find_text
# (Alternatively, apply phi map to all indexed items; here we combine both)
narrowed = subtractive_find_phi(current_items, find_text)
puts "--- Narrowed to #{narrowed.size} items via φ-crawl ---"
narrowed.each { |item| display_relevance(item) }
current_items = narrowed
else
items = @query_engine.query(input, mode: :latest)
puts "--- Found #{items.size} relevant items (others hidden) ---"
items.each { |item| display_relevance(item) }
current_items = items
end
end
end
private
def subtractive_find_phi(visible_items, substring)
# Use phi map to get candidates from the entire index
candidates = @phi_map.find_substring(substring, @embedder)
# Intersect with currently visible items (the subtractive set)
visible_ids = Set.new(visible_items.map(&:id))
candidates.select { |item| visible_ids.include?(item.id) }
end
def display_relevance(item)
time = item.meta[:timestamp]&.iso8601 || 'unknown'
snippet = item.content.length > 200 ? item.content[0..200] + '...' : item.content
puts "[#{time}] #{item.path.basename}: #{snippet}"
end
end
Oakmont Legal Authority
Lane County, Oregon
Copeland iNC.
01012026 122200
On Fri, Jul 17, 2026 at 5:30 AM Oakmont Copeland <oakmontcopeland@gmail.com> wrote:
Base O Converter
# phi_converter.rb
class PhiConverter
PHI = (1 + Math.sqrt(5)) / 2
# Convert a non-negative integer to a phinary string (finite representation).
# Uses greedy algorithm: subtract largest power of phi less than or equal to remainder.
def self.int_to_phinary(n)
return "0" if n == 0
# Precompute powers of phi until > n
powers = [1.0]
while powers.last * PHI <= n
powers << powers.last * PHI
end
powers.reverse!
digits = ""
remainder = n.to_f
powers.each do |p|
if remainder >= p
digits << "1"
remainder -= p
else
digits << "0"
end
end
digits.sub(/^0+/, "") # strip leading zeros
end
# Convert a decimal string (e.g. "1.6801") to an integer by treating it as a
# fixed-point number scaled by 10^8, then convert integer to phinary.
def self.decimal_to_phinary(decimal_str, scale = 10**8)
# parse as float, multiply by scale, round to integer
scaled = (decimal_str.to_f * scale).round
int_to_phinary(scaled)
end
end
# phinary_index.rb
class PhinaryIndex
Entry = Struct.new(:phinary, :item)
def initialize
@trie = {} # hash of character -> subtrie or entry
end
# Insert an item with its phinary equation
def add(item, phinary)
node = @trie
phinary.each_char do |ch|
node[ch] ||= {}
node = node[ch]
end
node[:__item__] = item
end
# Remove an item (traverse by its stored phinary)
def remove(item)
# We'd need to know the phinary; store it in the item meta or keep a reverse map.
# For brevity: assume we store the phinary string in item.meta[:phinary]
return unless item.meta[:phinary]
node = @trie
item.meta[:phinary].each_char do |ch|
break unless node[ch]
node = node[ch]
end
node.delete(:__item__)
end
# Return all items whose phinary string starts with the given prefix
def search_by_prefix(phinary_prefix)
node = @trie
phinary_prefix.each_char do |ch|
return [] unless node[ch]
node = node[ch]
end
collect_items(node)
end
private
def collect_items(node)
results = []
results << node[:__item__] if node[:__item__]
node.each do |key, child|
next if key == :__item__
results.concat(collect_items(child))
end
results
end
end
Oakmont Legal Authority
Lane County, Oregon
Copeland iNC.
01012026 122200
On Fri, Jul 17, 2026 at 5:28 AM Oakmont Copeland <oakmontcopeland@gmail.com> wrote:
class Application
attr_reader :config, :vindex, :query_engine, :llm, :indexer, :phi_converter, :phinary_index
def initialize(config: Config.new, embedder: nil, llm: nil, spatial: false)
# ... all previous initialization ...
@phi_converter = PhiConverter
@phinary_index = PhinaryIndex.new
end
# After indexing an item, also compute its phinary key and add to trie
def index_file(path, lat: nil, lng: nil)
# ... existing versioned add ...
item = @vindex.latest_items[file_id]
if item
# Compute phi coordinate (as before) and then its phinary representation
phi_val = @phi_map.phi_coordinate(item.embedding).abs.to_i # non-negative integer
phinary_str = PhiConverter.int_to_phinary(phi_val)
# Store the phinary in item's meta so we can remove later
item.meta[:phinary] = phinary_str
@phinary_index.add(item, phinary_str)
end
end
# New interactive loop: Sacred Key Revelation
def run_phinary_reveal
initial_scan
start_watcher
puts "Sacred Key mode. Enter a decimal key (like 1.6801) to reveal items."
puts "Type more digits to narrow, or 'clear' to hide all."
current_visible = [] # items currently shown
loop do
print "\n > "
input = gets&.chomp
break if input.nil? || input.downcase == 'exit'
if input.downcase == 'clear'
current_visible = []
puts "All items hidden."
else
# Interpret input as a decimal, convert to phinary prefix
begin
phinary_prefix = PhiConverter.decimal_to_phinary(input)
matched = @phinary_index.search_by_prefix(phinary_prefix)
puts "--- Revealed #{matched.size} items for key #{input} (phinary prefix: #{phinary_prefix}) ---"
matched.each { |item| display_relevance(item) }
current_visible = matched
rescue => e
puts "Invalid key: #{e.message}"
end
end
end
end
private
def display_relevance(item)
# same as before
time = item.meta[:timestamp]&.iso8601 || 'unknown'
snippet = item.content.length > 200 ? item.content[0..200] + '...' : item.content
puts "[#{time}] #{item.path.basename}: #{snippet}"
end
end
Oakmont Legal Authority
Lane County, Oregon
Copeland iNC.
01012026 122200
On Fri, Jul 17, 2026 at 5:28 AM Oakmont Copeland <oakmontcopeland@gmail.com> wrote:
Dockerfile
FROM ruby:3.3-slim
RUN apt-get update && apt-get install -y build-essential && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY Gemfile Gemfile.lock ./
RUN bundle install
COPY . .
RUN mkdir -p /root/.knowos
CMD ["ruby", "bin/knowos"]
Docker-compose.yml
version: '3.8'
services:
knowos:
build: .
container_name: knowos_os
volumes:
- ./lib:/app/lib
- ./bin:/app/bin
- ~/Documents:/data:ro # read-only for safety; change as needed
- knowos_index:/root/.knowos
environment:
KNOWOS_WATCH_DIRS: "/data"
KNOWOS_TOP_K: 5
KNOWOS_VECTOR_DIM: 512
stdin_open: true
tty: true
watcher:
image: alpine:latest
command: >
sh -c "apk add --no-cache watchexec &&
watchexec -r -w /app/lib -w /app/bin -- docker-compose restart knowos"
volumes:
- ./lib:/app/lib
- ./bin:/app/bin
- /var/run/docker.sock:/var/run/docker.sock
volumes:
knowos_index:
Gemfile
source 'https://rubygems.org'
gem 'listen', '~> 3.8'
gem 'minitest', '~> 5.18'
Oakmont Legal Authority
Lane County, Oregon
Copeland iNC.
01012026 122200
On Fri, Jul 17, 2026 at 5:27 AM Oakmont Copeland <oakmontcopeland@gmail.com> wrote:
/bin/knowos
#!/usr/bin/env ruby
require_relative '../lib/knowos'
app = KnowOS::Application.new
app.run_interactive # or app.run_subtractive_search or app.run_phinary_reveal
Oakmont Legal Authority
Lane County, Oregon
Copeland iNC.
01012026 122200
On Fri, Jul 17, 2026 at 5:26 AM Oakmont Copeland <oakmontcopeland@gmail.com> wrote:
require 'digest'
require 'fileutils'
require 'json'
require 'logger'
require 'set'
require 'time'
require 'pathname'
require 'listen'
module KnowOS
VERSION = '1.0.0'
# -----------------------------------------------------------------
# Configuration
# -----------------------------------------------------------------
class Config
attr_accessor :index_dir, :watch_dirs, :top_k_retrieval,
:graph_expansion_hops, :log_level, :vector_dim
def initialize(**options)
@index_dir = Pathname(options[:index_dir] || Dir.home + '/.knowos')
@watch_dirs = Array(options[:watch_dirs])&.map { |d| Pathname(d) } || [Pathname(Dir.home +
'/Documents')]
@top_k_retrieval = options[:top_k_retrieval] || 5
@graph_expansion_hops = options[:graph_expansion_hops] || 1
@log_level = options[:log_level] || Logger::INFO
@vector_dim = options[:vector_dim] || 512
end
def self.from_env
new(
index_dir: ENV['KNOWOS_INDEX_DIR'],
watch_dirs: ENV['KNOWOS_WATCH_DIRS']&.
split(':'),
top_k_retrieval: ENV['KNOWOS_TOP_K']&.to_i,
graph_expansion_hops: ENV['KNOWOS_GRAPH_HOPS']&.to_
i,
log_level: ENV['KNOWOS_LOG_LEVEL']&.to_i,
vector_dim: ENV['KNOWOS_VECTOR_DIM']&.to_i
)
end
def ensure_index_dir!
FileUtils.mkdir_p(@index_dir) unless @index_dir.exist?
end
end
def self.logger
@logger ||= begin
l = Logger.new($stdout)
l.level = config.log_level
l.formatter = proc { |severity, datetime, _, msg| "#{datetime} [#{severity}] KnowOS: #{msg}\n" }
l
end
end
def self.config
@config ||= Config.from_env
end
# ------------------------------
-----------------------------------
# KnowledgeItem
# -----------------------------------------------------------------
KnowledgeItem = Struct.new(:id, :path, :content, :embedding, :meta, keyword_init: true)
# ------------------------------
-----------------------------------
# MockEmbedder (deterministic, normalized)
# -----------------------------------------------------------------
class MockEmbedder
def initialize(dim: 512, seed: 42)
@dim = dim
@rng = Random.new(seed)
end
def encode(texts)
Array(texts).map do |text|
hash = Digest::SHA256.hexdigest(text.downcase)
seed = hash.to_i(16) % (2**31 - 1)
rng = Random.new(seed)
vec = Array.new(@dim) { rng.rand(-1.0..1.0) }
normalize(vec)
end
end
private
def normalize(vec)
norm = Math.sqrt(vec.sum { |x| x**2 })
norm.zero? ? vec : vec.map { |x| x / norm }
end
end
# -----------------------------------------------------------------
# VectorIndex (brute-force for demo, ready for HNSW)
# -----------------------------------------------------------------
class VectorIndex
def initialize(dim:, embedder:)
@dim = dim
@embedder = embedder
@items = []
@vectors = []
end
def add(item)
@items << item
@vectors << item.embedding
end
def remove(item_id)
idx = @items.index { |it| it.id == item_id }
return unless idx
@items.delete_at(idx)
@vectors.delete_at(idx)
end
def search(query_embedding, k:)
return [] if @vectors.empty?
k = [k, @vectors.size].min
similarities = @vectors.map { |v| dot(query_embedding, v) }
top = similarities.each_with_index
.sort_by { |sim, _| -sim }
.take(k)
.map { |_, i| @items[i] }
top
end
private
def dot(a, b)
a.zip(b).sum { |x, y| x * y }
end
end
# -----------------------------------------------------------------
# SpatialIndex (mock grid)
# -----------------------------------------------------------------
class SpatialIndex
def initialize(precision: 5)
@grid = {}
@precision = precision
end
def add(id, lat, lng)
cell = encode(lat, lng)
@grid[cell] ||= Set.new
@grid[cell] << id
end
def remove(id, lat, lng)
cell = encode(lat, lng)
@grid[cell]&.delete(id)
end
def query(lat, lng)
cell = encode(lat, lng)
@grid[cell]&.to_a || []
end
private
def encode(lat, lng)
lat_bin = ((lat + 90) * 1e5).to_i.to_s(2).rjust(20, '0')
lng_bin = ((lng + 180) * 1e5).to_i.to_s(2).rjust(20, '0')
interleaved = lat_bin.chars.zip(lng_bin.
chars).flatten.join[0...@precision]
interleaved
end
end
# ------------------------------
-----------------------------------
# GraphManager
# -----------------------------------------------------------------
class GraphManager
def initialize
@graph = {}
end
def update_edges(item_id, connected_ids)
@graph[item_id] ||= Set.new
@graph[item_id].merge(
connected_ids)
connected_ids.each do |cid|
@graph[cid] ||= Set.new
@graph[cid] << item_id
end
end
def remove_node(item_id)
return unless @graph.key?(item_id)
@graph[item_id].each { |n| @graph[n]&.delete(item_id) }
@graph.delete(item_id)
end
def expand(seed_ids, hops:)
current = Set.new(seed_ids)
visited = Set.new(seed_ids)
hops.times do
next_hop = Set.new
current.each do |node|
(@graph[node] || []).each do |neighbor|
unless visited.include?(neighbor)
visited << neighbor
next_hop << neighbor
end
end
end
current = next_hop
end
visited
end
end
# ------------------------------
-----------------------------------
# VersionedIndexV2 (latest + all versions, spatial)
# -----------------------------------------------------------------
class VersionedIndexV2
attr_reader :embedder
def initialize(dim:, embedder:, spatial: false)
@dim = dim
@embedder = embedder
@spatial = spatial
@items = {}
@latest_items = {}
@chains = {}
@all_index = VectorIndex.new(dim: dim, embedder: embedder)
@latest_index = VectorIndex.new(dim: dim, embedder: embedder)
@spatial_index = SpatialIndex.new if spatial
@graph_manager = GraphManager.new
end
def add_version(file_id:, content:, timestamp:, path:, lat: nil, lng: nil, embedding: nil)
version_id = "#{file_id}@#{timestamp.iso8601(3)}"
emb = embedding || @embedder.encode([content]).first
item = KnowledgeItem.new(
id: version_id,
path: path,
content: content,
embedding: emb,
meta: { file_id: file_id, timestamp: timestamp }
)
old_latest = @latest_items[file_id]
@latest_index.remove(old_latest.id) if old_latest
@items[version_id] = item
@all_index.add(item)
@latest_items[file_id] = item
@latest_index.add(item)
chain = @chains[file_id] ||= []
idx = chain.bsearch_index { |t| t >= timestamp } || chain.size
chain.insert(idx, timestamp)
@spatial_index.add(version_id, lat, lng) if @spatial && lat && lng
build_graph_edges(item)
item
end
def remove_file(file_id)
latest = @latest_items.delete(file_id)
@latest_index.remove(latest.id
) if latest
@chains.delete(file_id)
versions = @items.values.select { |it| it.meta[:file_id] == file_id }
versions.each { |v| @all_index.remove(v.id) }
@items.reject! { |id, _| id.start_with?(file_id + '@') }
@graph_manager.remove_node(
file_id)
end
def search_latest(query_emb, k:)
@latest_index.search(query_
emb, k: k)
end
def search_all(query_emb, k:, time_range: nil)
candidates = if time_range
@items.values.select { |it| time_range.cover?(it.meta[:
timestamp]) }
else
@items.values
end
return [] if candidates.empty?
similarities = candidates.map { |it| dot(query_emb, it.embedding) }
candidates.zip(similarities).
sort_by { |_, s| -s }.take(k).map(&:first)
end
def search_spatiotemporal(query_emb, lat:, lng:, time_range: nil, k:)
return [] unless @spatial
ids = @spatial_index.query(lat, lng)
candidates = ids.map { |id| @items[id] }.compact
candidates = candidates.select { |it| time_range.cover?(it.meta[:
timestamp]) } if time_range
return [] if candidates.empty?
similarities = candidates.map { |it| dot(query_emb, it.embedding) }
candidates.zip(similarities).
sort_by { |_, s| -s }.take(k).map(&:first)
end
def expand_graph(seed_items, hops: 1)
seed_ids = Set.new(seed_items.map(&:id))
expanded_ids = @graph_manager.expand(seed_
ids, hops: hops)
expanded_ids.map { |id| @items[id] }.compact
end
private
def dot(a, b); a.zip(b).sum { |x, y| x * y }; end
def build_graph_edges(item)
words = Set.new(item.content.downcase.
split)
@latest_items.each do |fid, other|
next if fid == item.meta[:file_id]
other_words = Set.new(other.content.
downcase.split)
if (words & other_words).size > 20
@graph_manager.update_edges(item.id, Set[other.id])
end
end
end
end
# ------------------------------
-----------------------------------
# File Utilities
# ------------------------------
-----------------------------------
module FileUtils2
def self.compute_file_id(path)
Digest::SHA256.hexdigest(path.
to_s)[0..15]
end
def self.extract_content(path)
File.read(path, encoding: 'UTF-8', invalid: :replace, undef: :replace, replace: '')
rescue => e
KnowOS.logger.warn "Cannot read #{path}: #{e.message}"
''
end
end
# ------------------------------
-----------------------------------
# VersionedIndexer (for watcher)
# -----------------------------------------------------------------
class VersionedIndexer
def initialize(versioned_index:, phi_map: nil)
@vindex = versioned_index
@phi_map = phi_map
end
def index_file(path, lat: nil, lng: nil)
content = FileUtils2.extract_content(
path)
return if content.empty?
timestamp = Time.now
file_id = FileUtils2.compute_file_id(
path)
item = @vindex.add_version(
file_id: file_id,
content: content,
timestamp: timestamp,
path: path,
lat: lat,
lng: lng
)
@phi_map.add(item) if @phi_map && item
KnowOS.logger.info "Versioned #{path} (latest: #{item&.id})"
end
def unindex_file(path)
file_id = FileUtils2.compute_file_id(
path)
@vindex.remove_file(file_id)
KnowOS.logger.info "Removed all versions of #{path}"
end
end
# ------------------------------
-----------------------------------
# FileWatcher
# -----------------------------------------------------------------
class FileWatcher
def initialize(watch_dirs, indexer)
@watch_dirs = watch_dirs.map(&:to_s)
@indexer = indexer
end
def start
listener = Listen.to(*@watch_dirs, only: /\.(txt|md|rb|py|log|tif|png|
jpg|jpeg)$/i) do |modified, added, removed|
added.each { |path| safe { @indexer.index_file(Pathname(path)) } }
modified.each { |path| safe { @indexer.index_file(Pathname(path)) } }
removed.each { |path| safe { @indexer.unindex_file(Pathname(path)) } }
end
listener.start
listener
end
private
def safe
yield
rescue => e
KnowOS.logger.error "Watcher error: #{e.message}"
end
end
# -----------------------------------------------------------------
# PhiNodeMap (golden-ratio based coordinate)
# -----------------------------------------------------------------
class PhiNodeMap
GOLDEN_ANGLE = 2.0 * Math::PI * (2.0 - 1.618033988749895)
def initialize(dim:)
@dim = dim
@entries = [] # sorted by phi_coordinate
end
def add(item)
n = phi_coordinate(item.embedding)
@entries << { n: n, item: item }
@entries.sort_by! { |e| e[:n] }
end
def remove(item_id)
@entries.reject! { |e| e[:item].id == item_id }
end
def find_substring(substring, embedder, radius: 1000)
sub_emb = embedder.encode([substring]).first
target_n = phi_coordinate(sub_emb)
left = @entries.bsearch_index { |e| e[:n] >= target_n - radius } || 0
right = @entries.bsearch_index { |e| e[:n] > target_n + radius } || @entries.size
results = []
(left...right).each do |i|
item = @entries[i][:item]
results << item if item.content.downcase.include?(substring.downcase)
end
results
end
def phi_coordinate(vector)
x = 0.0; y = 0.0
vector.each_with_index do |val, i|
theta = i * GOLDEN_ANGLE
radius = Math.sqrt(i.to_f / @dim)
x += val * radius * Math.cos(theta)
y += val * radius * Math.sin(theta)
end
(Math.atan2(y, x) * 1000 + Math.sqrt(x*x + y*y) * 1000000).to_i
end
end
# -----------------------------------------------------------------
# PhiConverter (integer to finite phinary string)
# -----------------------------------------------------------------
class PhiConverter
PHI = (1 + Math.sqrt(5)) / 2
def self.int_to_phinary(n)
return "0" if n == 0
powers = [1.0]
while powers.last * PHI <= n
powers << powers.last * PHI
end
powers.reverse!
digits = ""
remainder = n.to_f
powers.each do |p|
if remainder >= p
digits << "1"
remainder -= p
else
digits << "0"
end
end
digits.sub(/^0+/, "")
end
def self.decimal_to_phinary(
decimal_str, scale = 10**8)
scaled = (decimal_str.to_f * scale).round
int_to_phinary(scaled)
end
end
# ------------------------------
-----------------------------------
# PhinaryIndex (trie over phinary strings)
# -----------------------------------------------------------------
class PhinaryIndex
def initialize
@trie = {}
end
def add(item, phinary)
node = @trie
phinary.each_char do |ch|
node[ch] ||= {}
node = node[ch]
end
node[:__item__] = item
end
def remove(phinary)
node = @trie
phinary.each_char { |ch| break unless node[ch]; node = node[ch] }
node.delete(:__item__) if node
end
def search_by_prefix(phinary_
prefix)
node = @trie
phinary_prefix.each_char do |ch|
return [] unless node[ch]
node = node[ch]
end
collect_items(node)
end
private
def collect_items(node)
results = []
results << node[:__item__] if node[:__item__]
node.each do |key, child|
next if key == :__item__
results.concat(collect_items(
child))
end
results
end
end
# ------------------------------
-----------------------------------
# QueryEngine
# -----------------------------------------------------------------
class QueryEngine
def initialize(versioned_index:, config:)
@vindex = versioned_index
@config = config
end
def query(question, mode: :latest, time_range: nil, lat: nil, lng: nil, top_k: @config.top_k_retrieval)
q_emb = @vindex.embedder.encode([question]).first
items = case mode
when :latest
@vindex.search_latest(q_emb, k: top_k)
when :all
@vindex.search_all(q_emb, k: top_k, time_range: time_range)
when :spatiotemporal
@vindex.search_spatiotemporal(
q_emb, lat: lat, lng: lng, time_range: time_range, k: top_k)
else
@vindex.search_latest(q_emb, k: top_k)
end
expanded = @vindex.expand_graph(items, hops: @config.graph_expansion_hops)
seen = Set.new
final = []
items.each { |it| seen << it.id; final << it }
expanded.each { |it| final << it unless seen.include?(it.id); seen << it.id }
final
end
end
# ------------------------------
-----------------------------------
# MockLLM
# -----------------------------------------------------------------
class MockLLM
def answer(question:, items:)
context = items.first(10).map { |item| "[#{item.meta[:timestamp]&.
iso8601}] #{item.path.basename}: #{item.content[0..200]}" }.join("\n")
prompt = "CONTEXT:\n#{context}\n\
nQUESTION: #{question}\nANSWER:"
KnowOS.logger.info "LLM prompt (#{prompt.length} chars)"
"[Simulated] Based on #{items.size} relevant items."
end
end
# ------------------------------
-----------------------------------
# Application
# -----------------------------------------------------------------
class Application
attr_reader :config, :vindex, :query_engine, :llm, :indexer, :phi_map, :phinary_index
def initialize(config: Config.new, embedder: nil, llm: nil, spatial: false)
@config = config
@config.ensure_index_dir!
@embedder = embedder || MockEmbedder.new(dim: config.vector_dim)
@vindex = VersionedIndexV2.new(dim: config.vector_dim, embedder: @embedder, spatial: spatial)
@phi_map = PhiNodeMap.new(dim: config.vector_dim)
@indexer = VersionedIndexer.new(versioned_index: @vindex, phi_map: @phi_map)
@query_engine = QueryEngine.new(versioned_index: @vindex, config: config)
@llm = llm || MockLLM.new
@phinary_index = PhinaryIndex.new
end
def initial_scan
KnowOS.logger.info "Initial scan of #{@config.watch_dirs}"
@config.watch_dirs.each do |dir|
next unless dir.exist?
Pathname.glob(dir + '**/*').each do |path|
next unless path.file? && path.extname.match?(/\.(txt|md|rb|py|log|tif|png|jpg|jpeg)$/i)
index_file(path)
end
end
end
def start_watcher
@listener = FileWatcher.new(@config.watch_dirs, @indexer).start
KnowOS.logger.info "File watcher active"
end
def index_file(path, lat: nil, lng: nil)
@indexer.index_file(path, lat: lat, lng: lng)
# After indexing, also add to phinary index
file_id = FileUtils2.compute_file_id(path)
latest = @vindex.instance_variable_get(:@latest_items)[file_id]
if latest
phi_val = @phi_map.phi_coordinate(latest.embedding).abs
phinary = PhiConverter.int_to_phinary(phi_val)
latest.meta[:phinary] = phinary
@phinary_index.add(latest, phinary)
end
end
def answer(question, mode: :latest, time_range: nil, lat: nil, lng: nil)
items = @query_engine.query(question, mode: mode, time_range: time_range, lat: lat, lng: lng)
@llm.answer(question: question, items: items)
end
# Standard interactive loop
def run_interactive
initial_scan
start_watcher
puts "KnowOS ready. Ask anything (type 'exit' to quit)."
loop do
print "\n> "
input = gets&.chomp
break if input.nil? || input.downcase == 'exit'
puts answer(input)
end
@listener&.stop
end
# Subtractive search: everything hidden, then find: to narrow
def run_subtractive_search
initial_scan
start_watcher
puts "Subtractive Search. Enter a query to reveal relevant items; 'find: <text>' to narrow."
current = []
loop do
print "\n> "
input = gets&.chomp
break if input.nil? || input.downcase == 'exit'
if input.downcase.start_with?('find:')
text = input[5..].strip
narrowed = @phi_map.find_substring(text, @embedder)
visible_ids = Set.new(current.map(&:id))
narrowed = narrowed.select { |it| visible_ids.include?(it.id) }
puts "--- Narrowed to #{narrowed.size} items ---"
narrowed.each { |it| display(it) }
current = narrowed
else
items = @query_engine.query(input, mode: :latest)
puts "--- Found #{items.size} relevant items ---"
items.each { |it| display(it) }
current = items
end
end
@listener&.stop
end
# Phinary Key Revelation: enter decimal key to reveal items
def run_phinary_reveal
initial_scan
start_watcher
puts "Phinary Key mode. Enter a decimal key (e.g., 1.6801) to reveal items. 'clear' to hide all."
current = []
loop do
print "\n > "
input = gets&.chomp
break if input.nil? || input.downcase == 'exit'
if input.downcase == 'clear'
current = []
puts "All items hidden."
else
begin
phinary_prefix = PhiConverter.decimal_to_phinary(input)
matched = @phinary_index.search_by_prefix(phinary_prefix)
puts "--- Revealed #{matched.size} items for key #{input} ---"
matched.each { |it| display(it) }
current = matched
rescue => e
puts "Invalid key: #{e.message}"
end
end
end
@listener&.stop
end
private
def display(item)
time = item.meta[:timestamp]&.iso8601 || 'unknown'
snippet = item.content.length > 200 ? item.content[0..200] + '...' : item.content
puts "[#{time}] #{item.path.basename}: #{snippet}"
end
end
end
Oakmont Legal Authority
Lane County, Oregon
Copeland iNC.
01012026 122200
On Fri, Jul 17, 2026 at 5:26 AM Oakmont Copeland <oakmontcopeland@gmail.com> wrote:
class KnowPathManager
KNOWPATH_FILE = 'knowpath'
def initialize(storage_dir)
@dir = Pathname(storage_dir)
@dir.mkdir_p
@file = @dir / KNOWPATH_FILE
@raw = File.exist?(@file) ? File.read(@file).strip : ''
end
# Return the raw colon-separated string
def to_s
@raw
end
# Parse into typed components
def components
@raw.split(':').map do |part|
next nil if part.empty?
if part.start_with?('macro:')
{ type: :macro, value: part[6..] }
elsif part.start_with?('env:')
{ type: :env, value: part[4..] }
elsif part.start_with?('llm:')
{ type: :llm, value: part[4..] }
else
{ type: :path, value: part }
end
end.compact
end
# Resolve components into concrete lists of paths, macros, env vars, etc.
def resolve(env: ENV)
paths = []
macros = []
llm_model = nil
components.each do |comp|
case comp[:type]
when :path
paths << comp[:value]
when :macro
macros << comp[:value]
when :env
# Expand environment variable; its value might be a colon-separated list itself
expanded = env[comp[:value]] || ''
expanded.split(':').each { |p| paths << p unless p.empty? }
when :llm
llm_model = comp[:value]
end
end
{ paths: paths.uniq, macros: macros.uniq, llm_model: llm_model }
end
# Add a component to the string
def add(entry)
if @raw.empty?
@raw = entry
else
@raw += ':' + entry
end
save
end
# Remove a component (exact match)
def remove(entry)
parts = @raw.split(':')
parts.delete(entry)
@raw = parts.join(':')
save
end
# Clear all
def clear
@raw = ''
save
end
# Save to disk
def save
File.write(@file, @raw)
end
end
U
def apply_knowpath
resolved = @knowpath.resolve(env: ENV)
# Add KnowPath directories to watch list (union with config)
resolved[:paths].each do |dir|
@config.watch_dirs << Pathname(dir) unless @config.watch_dirs.include?(Pathname(dir))
end
# Pre-load macros from KnowPath (only if they exist in macro manager)
resolved[:macros].each do |macro_name|
unless @macro_manager.get(macro_name)
# Optionally create a placeholder? We'll just log a warning.
KnowOS.logger.warn "KnowPath macro '#{macro_name}' not found; ignored."
end
end
# Set LLM model if specified (for future real LLM backend)
if resolved[:llm_model]
KnowOS.logger.info "KnowPath sets LLM model to '#{resolved[:llm_model]}'"
# @llm_model = resolved[:llm_model]
end
end
Oakmont Legal Authority
Lane County, Oregon
Copeland iNC.
01012026 122200
On Fri, Jul 17, 2026 at 5:22 AM Oakmont Copeland <oakmontcopeland@gmail.com> wrote:
when /\Aknowpath\s+add\s+(.+)/
entry = $1.strip
@knowpath.add(entry)
puts "Added '#{entry}' to KnowPath."
apply_knowpath
when /\Aknowpath\s+remove\s+(.+)/
entry = $1.strip
@knowpath.remove(entry)
puts "Removed '#{entry}' from KnowPath."
apply_knowpath
when 'knowpath clear'
@knowpath.clear
puts "KnowPath cleared."
apply_knowpath
when 'knowpath save'
@knowpath.save
puts "KnowPath saved."
when 'knowpath'
puts "KnowPath: #{@knowpath}"
Oakmont Legal Authority
Lane County, Oregon
Copeland iNC.
01012026 122200
On Fri, Jul 17, 2026 at 5:22 AM Oakmont Copeland <oakmontcopeland@gmail.com> wrote:
#!/usr/bin/env ruby
# frozen_string_literal: true
require 'digest'
require 'fileutils'
require 'json'
require 'logger'
require 'set'
require 'time'
require 'pathname'
require 'listen'
module KnowOS
VERSION = '3.0.0'
# -----------------------------------------------------------------
# Configuration
# -----------------------------------------------------------------
class Config
attr_accessor :index_dir, :watch_dirs, :top_k_retrieval,
:graph_expansion_hops, :log_level, :vector_dim
def initialize(**options)
@index_dir = Pathname(options[:index_dir] || Dir.home + '/.knowos')
@watch_dirs = Array(options[:watch_dirs]).map { |d| Pathname(d) }
@watch_dirs = [Pathname(Dir.home + '/Documents')] if @watch_dirs.empty?
@top_k_retrieval = options[:top_k_retrieval] || 5
@graph_expansion_hops = options[:graph_expansion_hops] || 1
@log_level = options[:log_level] || Logger::INFO
@vector_dim = options[:vector_dim] || 512
end
def self.from_env
new(
index_dir: ENV['KNOWOS_INDEX_DIR'],
watch_dirs: ENV['KNOWOS_WATCH_DIRS']&.split(':'),
top_k_retrieval: ENV['KNOWOS_TOP_K']&.to_i,
graph_expansion_hops: ENV['KNOWOS_GRAPH_HOPS']&.to_i,
log_level: ENV['KNOWOS_LOG_LEVEL']&.to_i,
vector_dim: ENV['KNOWOS_VECTOR_DIM']&.to_i
)
end
def ensure_index_dir!
FileUtils.mkdir_p(@index_dir) unless @index_dir.exist?
end
end
def self.logger
@logger ||= begin
l = Logger.new($stdout)
l.level = config.log_level
l.formatter = proc { |severity, datetime, _, msg| "#{datetime} [#{severity}] KnowOS: #{msg}\n" }
l
end
end
def self.config
@config ||= Config.from_env
end
# -----------------------------------------------------------------
# KnowledgeItem
# -----------------------------------------------------------------
KnowledgeItem = Struct.new(:id, :path, :content, :embedding, :meta, keyword_init: true)
# -----------------------------------------------------------------
# Universal Language Encoder / Decoder
# -----------------------------------------------------------------
class UniversalEncoder
def initialize(dim: 512, seed: 42)
@dim = dim
@rng = Random.new(seed)
end
# Encode text in any language to a universal vector.
# This mock generates a deterministic vector from a normalized form
# (e.g., lowercased, stripped), simulating language invariance.
def encode(text)
# In production, this would use a multilingual model like LaBSE.
normalized = text.downcase.gsub(/[^a-z0-9\s]/, '').strip
hash = Digest::SHA256.hexdigest(normalized)
seed = hash.to_i(16) % (2**31 - 1)
rng = Random.new(seed)
vec = Array.new(@dim) { rng.rand(-1.0..1.0) }
normalize(vec)
end
private
def normalize(v)
norm = Math.sqrt(v.sum { |x| x**2 })
norm.zero? ? v : v.map { |x| x / norm }
end
end
class UniversalDecoder
# Decode a universal vector into natural language in the given target language.
# This mock returns a placeholder translation; in reality it would use
# a small generative model trained on universal embeddings.
def decode(vector, target_lang)
# Here we simply look for the nearest document (simulating retrieval-augmented translation)
# and then add a language tag. For a fully standalone demo, we can return
# a phrase indicating translation would happen.
"[#{target_lang}] Translated content would appear here."
end
end
# -----------------------------------------------------------------
# VectorIndex
# -----------------------------------------------------------------
class VectorIndex
def initialize(dim:, embedder:)
@dim = dim
@embedder = embedder
@items = []
@vectors = []
end
def add(item)
@items << item
@vectors << item.embedding
end
def remove(item_id)
idx = @items.index { |it| it.id == item_id }
return unless idx
@items.delete_at(idx)
@vectors.delete_at(idx)
end
def search(query_embedding, k:)
return [] if @vectors.empty?
k = [k, @vectors.size].min
similarities = @vectors.map { |v| dot(query_embedding, v) }
top = similarities.each_with_index
.sort_by { |sim, _| -sim }
.take(k)
.map { |_, i| @items[i] }
top
end
private
def dot(a, b) = a.zip(b).sum { |x, y| x * y }
end
# -----------------------------------------------------------------
# SpatialIndex (mock)
# -----------------------------------------------------------------
class SpatialIndex
def initialize(precision: 5)
@grid = {}
@precision = precision
end
def add(id, lat, lng)
cell = encode(lat, lng)
@grid[cell] ||= Set.new
@grid[cell] << id
end
def remove(id, lat, lng)
cell = encode(lat, lng)
@grid[cell]&.delete(id)
end
def query(lat, lng)
cell = encode(lat, lng)
@grid[cell]&.to_a || []
end
private
def encode(lat, lng)
lat_bin = ((lat + 90) * 1e5).to_i.to_s(2).rjust(20, '0')
lng_bin = ((lng + 180) * 1e5).to_i.to_s(2).rjust(20, '0')
interleaved = lat_bin.chars.zip(lng_bin.chars).flatten.join[0...@precision]
interleaved
end
end
# -----------------------------------------------------------------
# GraphManager
# -----------------------------------------------------------------
class GraphManager
def initialize
@graph = {}
end
def update_edges(item_id, connected_ids)
@graph[item_id] ||= Set.new
@graph[item_id].merge(connected_ids)
connected_ids.each do |cid|
@graph[cid] ||= Set.new
@graph[cid] << item_id
end
end
def remove_node(item_id)
return unless @graph.key?(item_id)
@graph[item_id].each { |n| @graph[n]&.delete(item_id) }
@graph.delete(item_id)
end
def expand(seed_ids, hops:)
current = Set.new(seed_ids)
visited = Set.new(seed_ids)
hops.times do
next_hop = Set.new
current.each do |node|
(@graph[node] || []).each do |neighbor|
unless visited.include?(neighbor)
visited << neighbor
next_hop << neighbor
end
end
end
current = next_hop
end
visited
end
end
# -----------------------------------------------------------------
# VersionedIndexV2
# -----------------------------------------------------------------
class VersionedIndexV2
attr_reader :embedder
def initialize(dim:, embedder:, spatial: false)
@dim = dim
@embedder = embedder
@spatial = spatial
@items = {}
@latest_items = {}
@chains = {}
@all_index = VectorIndex.new(dim: dim, embedder: embedder)
@latest_index = VectorIndex.new(dim: dim, embedder: embedder)
@spatial_index = SpatialIndex.new if spatial
@graph_manager = GraphManager.new
end
def add_version(file_id:, content:, timestamp:, path:, lat: nil, lng: nil, embedding: nil)
version_id = "#{file_id}@#{timestamp.iso8601(3)}"
emb = embedding || @embedder.encode(content)
item = KnowledgeItem.new(
id: version_id, path: path, content: content,
embedding: emb, meta: { file_id: file_id, timestamp: timestamp }
)
old_latest = @latest_items[file_id]
@latest_index.remove(old_latest.id) if old_latest
@items[version_id] = item
@all_index.add(item)
@latest_items[file_id] = item
@latest_index.add(item)
chain = @chains[file_id] ||= []
idx = chain.bsearch_index { |t| t >= timestamp } || chain.size
chain.insert(idx, timestamp)
@spatial_index.add(version_id, lat, lng) if @spatial && lat && lng
build_graph_edges(item)
item
end
def remove_file(file_id)
latest = @latest_items.delete(file_id)
@latest_index.remove(latest.id) if latest
@chains.delete(file_id)
versions = @items.values.select { |it| it.meta[:file_id] == file_id }
versions.each { |v| @all_index.remove(v.id) }
@items.reject! { |id, _| id.start_with?(file_id + '@') }
@graph_manager.remove_node(file_id)
end
def search_latest(query_emb, k:)
@latest_index.search(query_emb, k: k)
end
def search_all(query_emb, k:, time_range: nil)
candidates = if time_range
@items.values.select { |it| time_range.cover?(it.meta[:timestamp]) }
else
@items.values
end
return [] if candidates.empty?
similarities = candidates.map { |it| dot(query_emb, it.embedding) }
candidates.zip(similarities).sort_by { |_, s| -s }.take(k).map(&:first)
end
def search_spatiotemporal(query_emb, lat:, lng:, time_range: nil, k:)
return [] unless @spatial
ids = @spatial_index.query(lat, lng)
candidates = ids.map { |id| @items[id] }.compact
candidates = candidates.select { |it| time_range.cover?(it.meta[:timestamp]) } if time_range
return [] if candidates.empty?
similarities = candidates.map { |it| dot(query_emb, it.embedding) }
candidates.zip(similarities).sort_by { |_, s| -s }.take(k).map(&:first)
end
def expand_graph(seed_items, hops: 1)
seed_ids = Set.new(seed_items.map(&:id))
expanded_ids = @graph_manager.expand(seed_ids, hops: hops)
expanded_ids.map { |id| @items[id] }.compact
end
private
def dot(a, b) = a.zip(b).sum { |x, y| x * y }
def build_graph_edges(item)
words = Set.new(item.content.downcase.split)
@latest_items.each do |fid, other|
next if fid == item.meta[:file_id]
other_words = Set.new(other.content.downcase.split)
if (words & other_words).size > 20
@graph_manager.update_edges(item.id, Set[other.id])
end
end
end
end
# -----------------------------------------------------------------
# File Utilities
# -----------------------------------------------------------------
module FileUtils2
def self.compute_file_id(path)
Digest::SHA256.hexdigest(path.to_s)[0..15]
end
def self.extract_content(path)
File.read(path, encoding: 'UTF-8', invalid: :replace, undef: :replace, replace: '')
rescue => e
KnowOS.logger.warn "Cannot read #{path}: #{e.message}"
''
end
end
# -----------------------------------------------------------------
# VersionedIndexer
# -----------------------------------------------------------------
class VersionedIndexer
def initialize(versioned_index:, phi_map: nil)
@vindex = versioned_index
@phi_map = phi_map
end
def index_file(path, lat: nil, lng: nil)
content = FileUtils2.extract_content(path)
return if content.empty?
timestamp = Time.now
file_id = FileUtils2.compute_file_id(path)
item = @vindex.add_version(
file_id: file_id, content: content, timestamp: timestamp,
path: path, lat: lat, lng: lng
)
@phi_map.add(item) if @phi_map && item
KnowOS.logger.info "Versioned #{path} (latest: #{item&.id})"
end
def unindex_file(path)
file_id = FileUtils2.compute_file_id(path)
@vindex.remove_file(file_id)
KnowOS.logger.info "Removed all versions of #{path}"
end
end
# -----------------------------------------------------------------
# FileWatcher
# -----------------------------------------------------------------
class FileWatcher
def initialize(watch_dirs, indexer)
@watch_dirs = watch_dirs.map(&:to_s)
@indexer = indexer
end
def start
listener = Listen.to(*@watch_dirs, only: /\.(txt|md|rb|py|log|tif|png|jpg|jpeg)$/i) do |modified, added,
removed|
added.each { |p| safe { @indexer.index_file(Pathname(p)) } }
modified.each { |p| safe { @indexer.index_file(Pathname(p)) } }
removed.each { |p| safe { @indexer.unindex_file(Pathname(p)) } }
end
listener.start
listener
end
private
def safe
yield
rescue => e
KnowOS.logger.error "Watcher error: #{e.message}"
end
end
# -----------------------------------------------------------------
# PhiNodeMap
# -----------------------------------------------------------------
class PhiNodeMap
GOLDEN_ANGLE = 2.0 * Math::PI * (2.0 - 1.618033988749895)
def initialize(dim:)
@dim = dim
@entries = []
end
def add(item)
n = phi_coordinate(item.embedding)
@entries << { n: n, item: item }
@entries.sort_by! { |e| e[:n] }
end
def remove(item_id)
@entries.reject! { |e| e[:item].id == item_id }
end
def find_substring(substring, embedder, radius: 1000)
sub_emb = embedder.encode(substring)
target_n = phi_coordinate(sub_emb)
left = @entries.bsearch_index { |e| e[:n] >= target_n - radius } || 0
right = @entries.bsearch_index { |e| e[:n] > target_n + radius } || @entries.size
results = []
(left...right).each do |i|
item = @entries[i][:item]
results << item if item.content.downcase.include?(substring.downcase)
end
results
end
def phi_coordinate(vector)
x = 0.0; y = 0.0
vector.each_with_index do |val, i|
theta = i * GOLDEN_ANGLE
radius = Math.sqrt(i.to_f / @dim)
x += val * radius * Math.cos(theta)
y += val * radius * Math.sin(theta)
end
(Math.atan2(y, x) * 1000 + Math.sqrt(x*x + y*y) * 1000000).to_i
end
end
# -----------------------------------------------------------------
# PhiConverter
# -----------------------------------------------------------------
class PhiConverter
PHI = (1 + Math.sqrt(5)) / 2
def self.int_to_phinary(n)
return "0" if n == 0
powers = [1.0]
while powers.last * PHI <= n
powers << powers.last * PHI
end
powers.reverse!
digits = ""
remainder = n.to_f
powers.each do |p|
if remainder >= p
digits << "1"
remainder -= p
else
digits << "0"
end
end
digits.sub(/^0+/, "")
end
def self.decimal_to_phinary(decimal_str, scale = 10**8)
scaled = (decimal_str.to_f * scale).round
int_to_phinary(scaled)
end
end
# -----------------------------------------------------------------
# PhinaryIndex
# -----------------------------------------------------------------
class PhinaryIndex
def initialize
@trie = {}
end
def add(item, phinary)
node = @trie
phinary.each_char do |ch|
node[ch] ||= {}
node = node[ch]
end
node[:__item__] = item
end
def remove(phinary)
node = @trie
phinary.each_char { |ch| break unless node[ch]; node = node[ch] }
node.delete(:__item__) if node
end
def search_by_prefix(phinary_prefix)
node = @trie
phinary_prefix.each_char do |ch|
return [] unless node[ch]
node = node[ch]
end
collect_items(node)
end
private
def collect_items(node)
results = []
results << node[:__item__] if node[:__item__]
node.each do |key, child|
next if key == :__item__
results.concat(collect_items(child))
end
results
end
end
# -----------------------------------------------------------------
# Macro & MacroManager
# -----------------------------------------------------------------
Macro = Struct.new(:name, :macro_type, :query_text, :file_path, :created_at, keyword_init: true)
class MacroManager
MACRO_FILE = 'macros.json'
def initialize(storage_dir)
@dir = Pathname(storage_dir)
@dir.mkdir_p
@file = @dir / MACRO_FILE
@macros = load
@query_counts = Hash.new(0)
end
def load
return {} unless @file.exist?
data = JSON.parse(File.read(@file))
data.map { |name, h| [name, Macro.new(h.transform_keys(&:to_sym))] }.to
'''
