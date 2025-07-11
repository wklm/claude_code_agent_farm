# Claude Code Agent Farm v2.0 - Docker Setup with Flutter 🐳🚜

This Docker setup provides a complete, containerized environment for running Claude Code Agent Farm v2.0 with Flutter development support. The new version includes automatic API quota handling and enhanced monitoring capabilities.

## Features

- 🐳 **Complete Flutter Development Environment**: Ubuntu 22.04 with Flutter SDK, Android SDK, and all dependencies
- 🤖 **Claude Code CLI Pre-installed**: Your Claude configuration is copied into the container
- 🚀 **Single Agent Per Container**: Clean isolation with 1 agent per container
- 📦 **Background Mode**: Spawn multiple containers working in parallel
- 📝 **Timestamped Logging**: All output includes timestamps in background mode
- 🔗 **Easy Attachment**: Connect to any running container to see live tmux session
- 🎯 **Simple Commands**: Bash aliases for common operations
- ⏸️ **Automatic API Quota Handling**: Uses `claude-auto-resume` to wait and resume when limits are hit
- 📊 **Enhanced Monitoring**: Integrated `claude-code-generic-hooks` for detailed metrics
- 🎼 **Simplified tmux Management**: Uses `tmux-composer` for cleaner session handling

## Prerequisites

- Docker installed and running
- Claude Code CLI configured locally (`~/.claude.json`)
- A Flutter project to work on

## Quick Start

### 1. Build the Docker Image

```bash
./build-docker.sh
```

This script:
- Copies your Claude configuration (`~/.claude.json`)
- Builds the Docker image with tag `claude-code-agent-farm:flutter`
- Includes Flutter SDK, Android SDK, Python 3.11+, and all dependencies
- Installs claude-auto-resume for automatic API quota handling
- Installs tmux-composer for simplified session management
- Installs claude-code-generic-hooks for monitoring

### 2. Run a Single Container (Interactive Mode)

```bash
# From your Flutter project directory
./run-docker.sh "Fix all type errors"

# Or with a prompt file
./run-docker.sh tasks.txt
```

### 3. Run Multiple Containers (Background Mode)

```bash
# Spawn 5 containers, each with 1 agent
./run-docker.sh -b 5 "Implement Flutter best practices"

# View logs with timestamps
docker logs -f ccfarm-1
docker logs -f ccfarm-2

# Attach to container 3 to see tmux session
./docker-attach.sh 3
```

## Bash Aliases

Add these convenient aliases by sourcing your bashrc:

```bash
source ~/.bashrc
```

Then use:

```bash
ccfarm              # Run single container interactively
ccfarm-bg           # Run in background mode
ccfarm-attach       # Attach to a running container
ccfarm-logs         # View logs of first container
ccfarm-stop         # Stop all containers
```

### Examples with Aliases

```bash
# Single container
ccfarm "Fix all Flutter type errors"

# Multiple containers in background
ccfarm-bg 10 "Implement comprehensive error handling"

# Attach to container 5
ccfarm-attach 5

# Stop all running containers
ccfarm-stop
```

## Container Architecture

### Directory Structure
- `/app` - Claude Code Agent Farm application
- `/workspace` - Your mounted project directory
- `/opt/flutter` - Flutter SDK installation
- `/opt/android-sdk` - Android SDK installation
- `/opt/claude-auto-resume` - Automatic API quota handling
- `/opt/claude-hooks` - Enhanced monitoring hooks
- `~/.config/claude-code/hooks` - User-specific hook configurations

### Environment
- **OS**: Ubuntu 22.04 LTS
- **Python**: 3.11+ with virtual environment
- **Flutter**: Latest stable channel with web support
- **Android**: SDK 33, Build Tools 33.0.0, Java 17
- **Claude Code**: Pre-installed with your configuration
- **claude-auto-resume**: Automatically handles API quota limits
- **tmux-composer**: Simplified tmux session management
- **claude-code-generic-hooks**: Advanced monitoring and metrics

## Advanced Usage

### Background Mode Details

When running in background mode (`-b N`):
- Creates N containers named `ccfarm-1`, `ccfarm-2`, ... `ccfarm-N`
- Each container runs exactly 1 agent
- All work on the same project directory (mounted volume)
- All use the same prompt
- Output includes timestamps and container numbers
- Containers run detached (in background)

### Viewing Progress

1. **Real-time Logs**:
   ```bash
   # Single container
   docker logs -f ccfarm-1
   
   # Multiple containers in parallel
   docker logs -f ccfarm-1 & docker logs -f ccfarm-2 & docker logs -f ccfarm-3
   ```

2. **Attach to Container**:
   ```bash
   ./docker-attach.sh 2
   ```
   - Shows the tmux session with the agent
   - Press `Ctrl+P Ctrl+Q` to detach without stopping
   - Press `Ctrl+C` to stop the agent and container

3. **Monitor All Containers**:
   ```bash
   docker ps --filter name=ccfarm-
   ```

### Managing Containers

```bash
# Stop specific container
docker stop ccfarm-3

# Stop all containers
docker stop $(docker ps -q --filter name=ccfarm-)

# Remove stopped containers
docker rm $(docker ps -aq --filter name=ccfarm-)

# Stop and remove all containers with one command
./ccfarm-stopremove.sh

# View container resource usage
docker stats --no-stream $(docker ps -q --filter name=ccfarm-)
```

The `ccfarm-stopremove.sh` script provides a convenient way to clean up all Claude Agent Farm containers at once. It will:
- Stop all running ccfarm-* containers
- Remove all ccfarm-* containers (both running and stopped)
- Show a summary of what was cleaned up

## Customization

### Using Different Configurations

```bash
# Mount custom config
docker run -it -v $(pwd):/workspace \
  -v /path/to/custom-config.json:/app/configs/custom.json \
  claude-code-agent-farm:flutter \
  --config /app/configs/custom.json \
  --agents 1
```

### Environment Variables

The container supports these environment variables:
- `PROMPT_TEXT` - Direct prompt text
- `PROMPT_FILE` - Path to prompt file
- `CONFIG_FILE` - Configuration file path
- `AGENTS` - Number of agents (always 1 in this setup)
- `AUTO_RESTART` - Enable auto-restart (default: true)

## What's New in v2.0

### Automatic API Quota Handling
When Claude's API quota is exhausted, the system automatically:
1. Detects the usage limit message
2. Extracts the reset time
3. Waits with a countdown timer
4. Resumes execution automatically

No manual intervention needed! Just let it run.

### Enhanced Monitoring
Each agent now has:
- Command execution tracking
- Performance metrics collection
- Automatic error recovery
- Context usage monitoring
- Detailed logs in `/workspace/.agent_logs/`

### Simplified Architecture
- 52% less code through modular design
- Leverages proven community tools
- Cleaner, more maintainable structure

## Migrating from v1.x

If you're upgrading from v1.x:

1. **Rebuild the Docker image** - New dependencies are included:
   ```bash
   ./build-docker.sh
   ```

2. **No configuration changes needed** - Your existing configs work as-is

3. **Automatic benefits**:
   - API quota handling works out of the box
   - Better monitoring with no extra setup
   - Same commands, better results

4. **New directories created**:
   - `.agent_logs/` - Detailed execution logs
   - `.agent_metrics/` - Performance metrics
   - `.claude_hooks/` - Hook configurations

The main difference is that agents will now automatically wait and resume when hitting API limits, instead of failing.

## Troubleshooting

### Container Won't Start
- Check if the image was built: `docker images | grep claude-code-agent-farm`
- Ensure Docker daemon is running: `docker ps`
- Check for port conflicts if applicable

### Claude Code Issues
- Verify your `~/.claude.json` exists before building
- Rebuild image if configuration changed: `./build-docker.sh`
- Check container logs for authentication errors
- API quota handling is automatic - just wait if limits are hit

### Flutter/Android Issues
- Flutter doctor output is shown on container start
- Android licenses are pre-accepted during build
- Web support is enabled by default

### Attachment Issues
- Ensure container is running: `docker ps`
- Use correct container number with `docker-attach.sh`
- If tmux session doesn't exist, agent may have completed

### Monitoring Issues
- Check `.agent_logs/` directory for detailed logs
- Metrics are stored in `.agent_metrics/`
- Hook configurations are in `.claude_hooks/`

### Executing Commands in Containers
When you need to run commands inside a container (e.g., for debugging), always exec as the workspace user, not root:

```bash
# Wrong - will run as root and Claude will refuse --dangerously-skip-permissions
docker exec -it ccfarm-1 bash

# Correct - runs as the workspace user
docker exec -it -u 1000:1000 ccfarm-1 bash

# To run Claude directly
docker exec -u 1000:1000 ccfarm-1 claude --version
```

The container automatically switches to a non-root user matching your workspace ownership to avoid permission issues.

## Performance Considerations

- Each container uses ~500MB RAM for the agent
- Flutter/Android tools add ~2GB to image size
- Consider system resources when spawning many containers
- Use `docker stats` to monitor resource usage

## Security Notes

- Containers run as non-root user `claude`
- Project directory is mounted read-write
- Claude configuration is copied (not mounted) for isolation
- Containers are isolated from each other

## Example Workflows

### Parallel Bug Fixing
```bash
# Run 10 agents to fix different bugs
ccfarm-bg 10 "Fix all type-check and lint errors in the codebase"

# Monitor progress
watch 'docker ps --filter name=ccfarm- --format "table {{.Names}}\t{{.Status}}"'
```

### Iterative Development
```bash
# Start with analysis
ccfarm "Analyze codebase and create improvement plan"

# Then implement in parallel
ccfarm-bg 5 "Implement the improvements from @IMPROVEMENT_PLAN.md"
```

### Continuous Integration
```bash
# In CI pipeline
docker run --rm -v $(pwd):/workspace \
  -e PROMPT_TEXT="Fix all failing tests" \
  claude-code-agent-farm:flutter \
  --agents 1 --no-monitor
```

## Contributing

To contribute Docker improvements:

1. Fork the repository
2. Create a feature branch
3. Test thoroughly with various Flutter projects
4. Submit a pull request

## Support

For issues specific to Docker setup:
- Check this documentation first
- Review container logs
- Open an issue with details about your environment

---

Happy farming with Docker and automatic API quota handling! 🐳🚜🤖⏸️