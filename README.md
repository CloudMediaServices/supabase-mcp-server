# Supabase MCP Server

A Model Context Protocol (MCP) server that provides seamless integration between AI assistants and Supabase databases. This server enables LLMs to perform CRUD operations on any Supabase database through standardized, well-documented tools.

## 🚀 Features

- **Full CRUD Operations**: Read, Create, Update, and Delete records in any Supabase table
- **Advanced Filtering**: Support for complex queries with multiple filter conditions
- **Safety First**: Built-in safety checks for destructive operations
- **Type Safety**: Full type hints and Pydantic validation
- **Comprehensive Error Handling**: Detailed error messages and logging
- **Flexible Querying**: Support for pagination, ordering, and column selection
- **Upsert Support**: Insert or update records in a single operation

## 📋 Prerequisites

- Python 3.11 or higher
- A Supabase project (self-hosted or cloud)
- Supabase service role key with appropriate permissions

## 🛠️ Installation

1. **Clone or download the project**:
   ```bash
   git clone <repository-url>
   cd supabase-mcp-server
   ```

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up environment variables**:
   ```bash
   cp .env.example .env
   # Edit .env with your Supabase credentials
   ```

4. **Configure your environment**:
   Edit the `.env` file with your Supabase credentials:
   ```env
   SUPABASE_URL=https://your-project-id.supabase.co
   SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here
   ```

## 🔧 Configuration

### Environment Variables

Edit `.env` file with your configuration:

```env
# Required: Your existing Supabase instance
SUPABASE_URL=https://your-project-id.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here

# Optional: Server configuration
LOG_LEVEL=INFO
```

### Finding Your Supabase Credentials

1. Go to your Supabase project dashboard
2. Navigate to **Settings** → **API**
3. Copy the **Project URL** for `SUPABASE_URL`
4. Copy the **service_role secret** for `SUPABASE_SERVICE_ROLE_KEY`

⚠️ **Important**: Use the `service_role` key, not the `anon` key, as it has full database access.

## 📁 GitHub Repository Setup

🔗 **[Complete GitHub Setup & SSH Deployment Guide](GITHUB_SETUP.md)**

Quick setup:
1. Create GitHub repository
2. Push code: `git init && git add . && git commit -m "Initial commit" && git push`
3. SSH deploy: `git clone YOUR_REPO && cd PROJECT && ./scripts/deploy.sh`

💡 **[SSH Quick Deploy Reference](SSH_QUICK_DEPLOY.md)** - 3-command deployment

## 🚀 Usage

### Local Development

#### Running the Server

```bash
python src/server.py
```

The server will start with stdio transport, which is the standard for MCP servers.

#### Installing in Claude Desktop

1. Add to your Claude Desktop MCP configuration:
   ```json
   {
     "servers": {
       "supabase": {
         "command": "python",
         "args": ["path/to/supabase-mcp-server/src/server.py"],
         "env": {
           "SUPABASE_URL": "https://your-project-id.supabase.co",
           "SUPABASE_SERVICE_ROLE_KEY": "your-service-role-key"
         }
       }
     }
   }
   ```

#### Using with MCP Inspector

For development and testing:
```bash
npx @modelcontextprotocol/inspector python src/server.py
```

### 🐳 Cloud Deployment

Deploy to your cloud Docker instance in minutes!

#### Quick Deploy (Automated)

1. **Upload files to your cloud server**:
   ```bash
   scp -r supabase-mcp-server/ user@your-server.com:/home/user/
   ```

2. **SSH and deploy**:
   ```bash
   ssh user@your-server.com
   cd supabase-mcp-server
   chmod +x scripts/deploy.sh
   ./scripts/deploy.sh
   ```

#### Manual Deploy

```bash
# Configure environment
cp .env.production .env
nano .env  # Add your Supabase credentials

# Deploy MCP server only (recommended)
docker-compose -f docker-compose.mcp-only.yml up -d

# Or deploy full stack (includes self-hosted Supabase)
docker-compose up -d
```

**Access**: Server available on port 8080

📖 **[Complete Deployment Guide](DEPLOYMENT.md)** - Includes security, monitoring, scaling, and troubleshooting

## 🔨 Available Tools

### 1. Read Table Rows
Query data from any table with filtering, ordering, and pagination.

**Usage**: "Show me all users where status is 'active'"
```json
{
  "table_name": "users",
  "filters": [{"column": "status", "operator": "eq", "value": "active"}],
  "order_by": "created_at",
  "limit": 10
}
```

### 2. Create Table Records
Insert new records into any table, with optional upsert functionality.

**Usage**: "Create a new user with name 'John' and email 'john@example.com'"
```json
{
  "table_name": "users",
  "records": [{"name": "John", "email": "john@example.com"}]
}
```

### 3. Update Table Records
Modify existing records based on specified conditions.

**Usage**: "Update the status to 'completed' for task with id 123"
```json
{
  "table_name": "tasks",
  "set_data": {"status": "completed"},
  "where_conditions": [{"column": "id", "operator": "eq", "value": 123}]
}
```

### 4. Delete Table Records
Remove records from tables with safety checks and confirmation.

**Usage**: "Delete all inactive users created before 2023"
```json
{
  "table_name": "users",
  "where_conditions": [
    {"column": "status", "operator": "eq", "value": "inactive"},
    {"column": "created_at", "operator": "lt", "value": "2023-01-01"}
  ],
  "confirm_delete": true
}
```

## 🔍 Filter Operators

The server supports various filter operators for precise querying:

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equal to | `{"column": "status", "operator": "eq", "value": "active"}` |
| `neq` | Not equal to | `{"column": "status", "operator": "neq", "value": "deleted"}` |
| `gt` | Greater than | `{"column": "age", "operator": "gt", "value": 18}` |
| `gte` | Greater than or equal | `{"column": "score", "operator": "gte", "value": 80}` |
| `lt` | Less than | `{"column": "price", "operator": "lt", "value": 100}` |
| `lte` | Less than or equal | `{"column": "discount", "operator": "lte", "value": 50}` |
| `like` | Pattern matching (case-sensitive) | `{"column": "name", "operator": "like", "value": "%john%"}` |
| `ilike` | Pattern matching (case-insensitive) | `{"column": "email", "operator": "ilike", "value": "%@gmail.com"}` |
| `in` | Value in list | `{"column": "category", "operator": "in", "value": ["tech", "science"]}` |
| `is` | Is null/true/false | `{"column": "deleted_at", "operator": "is", "value": null}` |

## 🧪 Testing

Run the test suite:
```bash
pytest tests/
```

Run with coverage:
```bash
pytest tests/ --cov=src
```

## 🛡️ Security Features

- **Environment Variable Validation**: Ensures required credentials are set
- **Input Validation**: Pydantic models validate all input data
- **Safety Checks**: Requires confirmation for destructive operations
- **Where Clause Requirements**: Updates and deletes require explicit conditions
- **Error Handling**: Comprehensive error handling with detailed logging

## 📁 Project Structure

```
supabase-mcp-server/
├── src/
│   └── server.py           # Main MCP server implementation
├── tests/
│   └── test_server.py      # Comprehensive test suite
├── requirements.txt        # Python dependencies
├── .env.example           # Environment variables template
├── README.md              # This file
├── PLANNING.md            # Project planning and architecture
├── TASK.md               # Task breakdown and progress
└── GLOBAL_RULES.md       # Development rules and standards
```

## 🔄 Example Usage Scenarios

### Scenario 1: Content Management
"Show me all published blog posts from this year, ordered by publication date"

### Scenario 2: User Management
"Create a new admin user and update their permissions"

### Scenario 3: Data Cleanup
"Find and delete all expired session tokens"

### Scenario 4: Analytics
"Get user count by registration month for the past year"

## 🐛 Troubleshooting

### Common Issues

1. **Environment Variables Not Set**
   - Error: "Missing environment variables"
   - Solution: Ensure `.env` file exists with correct `SUPABASE_URL` and `SUPABASE_SERVICE_ROLE_KEY`

2. **Database Connection Failed**
   - Error: "Failed to initialize Supabase client"
   - Solution: Verify your Supabase URL and service role key are correct

3. **Permission Denied**
   - Error: Various permission-related errors
   - Solution: Ensure your service role key has appropriate permissions for the tables you're accessing

4. **Table Not Found**
   - Error: Table-specific errors
   - Solution: Verify the table name exists in your Supabase database

## 📚 Development

### Code Style
- Follow PEP 8 standards
- Use type hints for all functions
- Include comprehensive docstrings
- Maximum 500 lines per file

### Testing Requirements
- Minimum 95% test coverage
- Test all CRUD operations
- Include edge cases and error scenarios
- Use pytest for all tests

## 🤝 Contributing

1. Follow the global rules defined in `GLOBAL_RULES.md`
2. Ensure all tests pass before submitting changes
3. Update documentation for any new features
4. Add appropriate error handling and logging

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- Built with the [Model Context Protocol](https://modelcontextprotocol.io/)
- Uses the [Supabase Python Client](https://github.com/supabase/supabase-py)
- Based on [FastMCP](https://github.com/modelcontextprotocol/python-sdk) framework
