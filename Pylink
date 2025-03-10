#!/bin/bash
set -euxo pipefail  # Exit on error, print commands, and fail on pipe errors
 
echo "Checking for virtual environment..."
if [ -d "venv" ]; then
    echo "Activating virtual environment..."
    source venv/bin/activate
else
    echo "No virtual environment found. Using system Python."
fi
 
echo "Installing dependencies..."
pip install -r requirements.txt
 
echo "Running Pylint for code quality check..."
pylint --fail-under=8 $(find . -name "*.py")  # Scan all Python files
 
echo "Building Docker image for testing..."
docker build -t test-image:latest .
 
echo "All tasks completed successfully!"
