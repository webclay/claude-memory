Kill all running Node.js and development server processes.

## Steps

1. Run `pkill -f node` to kill all Node processes
2. Also kill processes on common dev ports if needed:
   - Port 3000 (Next.js, React)
   - Port 3001 (alternate)
   - Port 5173 (Vite)
   - Port 8080 (common dev server)
3. Confirm which processes were killed
4. Ready for clean `npm run dev` or `pnpm dev`

## Commands to Run

```bash
# Kill all node processes
pkill -f node 2>/dev/null || true

# Alternative: kill by port if needed
lsof -ti:3000 | xargs kill -9 2>/dev/null || true
lsof -ti:5173 | xargs kill -9 2>/dev/null || true
```

## Output Format

```
PROCESSES TERMINATED

Killed:
- Node processes: [count]
- Port 3000: [cleared/not in use]
- Port 5173: [cleared/not in use]

Ready for: npm run dev / pnpm dev
```
