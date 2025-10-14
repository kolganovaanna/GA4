git init
echo "results/" > .gitignore
echo "data/" >> .gitignore
echo "scripts/" >> .gitignore
git add .gitignore
git commit -m "Add a Gitignore file"

sbatch --account=PAS2880 scripts/trimgalore.sh