# Draw Lots Static

A simple web application for drawing random numbers with a beautiful UI.

## Features

- Draw random numbers from a specified range
- Keep track of drawn numbers
- Clear history functionality
- Responsive design
- Beautiful animations

## Development

```bash
# Install dependencies
npm install

# Start development server
npm run serve

# Build for production
npm run build
```

## Deployment to GitHub Pages

1. Create a new repository on GitHub named `draw-lots-static`

2. Initialize your local repository and push to GitHub:
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<YOUR-USERNAME>/draw-lots-static.git
git push -u origin main
```

3. Make the deployment script executable:
```bash
chmod +x deploy.sh
```

4. Edit the `deploy.sh` file and replace `<USERNAME>` with your GitHub username

5. Run the deployment script:
```bash
./deploy.sh
```

6. Go to your repository settings on GitHub:
   - Navigate to "Settings" > "Pages"
   - Under "Source", select "gh-pages" branch
   - Click "Save"

Your site will be available at: `https://<YOUR-USERNAME>.github.io/draw-lots-static/`

## License

MIT 