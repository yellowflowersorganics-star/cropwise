# Contributing to CropWise

Thank you for your interest in contributing to CropWise! This document provides guidelines and instructions for contributing.

## 🤝 Code of Conduct

We are committed to providing a welcoming and inclusive environment. Please be respectful and constructive in all interactions.

## 🌟 How to Contribute

### Reporting Bugs

1. Check existing issues to avoid duplicates
2. Use the bug report template
3. Include:
   - Clear description
   - Steps to reproduce
   - Expected vs actual behavior
   - Environment details
   - Screenshots if applicable

### Suggesting Features

1. Check existing feature requests
2. Open a new issue with "Feature Request" label
3. Describe:
   - Use case
   - Proposed solution
   - Alternative approaches
   - Impact on existing features

### Contributing Code

1. **Fork the repository**
   ```bash
   git clone https://github.com/yourusername/cropwise.git
   cd cropwise
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**
   - Follow coding standards
   - Add tests if applicable
   - Update documentation

4. **Commit your changes**
   ```bash
   git add .
   git commit -m "feat: add crop recipe validation"
   ```

5. **Push and create PR**
   ```bash
   git push origin feature/your-feature-name
   ```

## 📝 Commit Message Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes (formatting)
- `refactor:` Code refactoring
- `test:` Adding or updating tests
- `chore:` Maintenance tasks

Examples:
```
feat: add mushroom crop recipe template
fix: correct humidity sensor reading calculation
docs: update ESP32 setup instructions
```

## 🏗️ Development Setup

### Backend Development

```bash
cd backend
npm install
npm run dev
```

### Frontend Development

```bash
cd frontend
npm install
npm run dev
```

### ESP32 Firmware

```bash
cd edge/esp32
pio run
```

## 🧪 Testing

### Backend Tests

```bash
cd backend
npm test
npm run test:coverage
```

### Frontend Tests

```bash
cd frontend
npm test
```

### Integration Tests

```bash
npm run test:integration
```

## 📐 Coding Standards

### JavaScript/TypeScript

- Use ESLint configuration
- Follow Airbnb style guide
- Prefer functional components (React)
- Use meaningful variable names
- Add JSDoc comments for functions

### C++ (ESP32)

- Follow Arduino style guide
- Use descriptive names
- Add header comments
- Keep functions focused

### Database

- Use migrations for schema changes
- Never commit secrets
- Follow naming conventions
- Add indexes for performance

## 📚 Documentation

- Update README when adding features
- Document APIs with examples
- Include inline code comments
- Update architecture diagrams if needed

## 🎨 UI/UX Guidelines

- Follow existing design patterns
- Ensure responsive design
- Test on multiple devices
- Maintain accessibility standards
- Use Tailwind utility classes

## 🔍 Code Review Process

1. All PRs require at least one approval
2. CI/CD must pass
3. No merge conflicts
4. Documentation updated
5. Tests added/updated

## 🏷️ Issue Labels

- `bug`: Something isn't working
- `feature`: New feature request
- `enhancement`: Improvement to existing feature
- `documentation`: Documentation improvements
- `good first issue`: Good for newcomers
- `help wanted`: Extra attention needed
- `priority: high`: Urgent issues

## 🌱 Contributing Crop Recipes

We welcome new crop recipes!

1. Create JSON file in `shared/examples/`
2. Follow schema: `shared/schemas/crop-recipe.schema.json`
3. Include:
   - All growth stages
   - Accurate environmental parameters
   - Expected yield data
   - Growing notes
4. Test recipe if possible
5. Submit PR with:
   - Recipe file
   - Documentation
   - Photos (optional)

## 🏆 Recognition

Contributors will be:
- Listed in CONTRIBUTORS.md
- Mentioned in release notes
- Credited in documentation

## 📞 Getting Help

- **Discord**: https://discord.gg/cropwise
- **GitHub Discussions**: https://github.com/yellowflowers/cropwise/discussions
- **Email**: dev@yellowflowers.tech

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for making CropWise better! 🌱**

