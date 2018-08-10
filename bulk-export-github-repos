#!/bin/sh
#/ Usage: bulk-export-github-repos organization-name filename.csv
#/
#/ Bulk exports a list of Git repositories from an organization
#/ given a csv file in the running directory
#/ containing the following data:
#/ [Source Repository Name]
#/
#/ Prior to running this script, a user should:
#/ - Create a personal access token, assigned to a scope of `admin:org` on the destination GHE instance, and store it as the environment variable, "GITHUB_TOKEN"
#/
#/ Example:
#/
#/   Given a filename.csv containing:
#/   reponame1
#/   reponame2
#/   reponame3
#/
#/   Run the script:
#/
#/   bulk-export-github-repos organization-name filename.csv

org=$1
filename=$2

ALL_REPOS="$(awk -v var="${org}" '{print "\""var, $1"\""}' OFS=/ ORS=", " ${filename} | sed 's/, $//g')"

# Run cURL command to start an export job of the repositories on GitHub.com
  RESPONSE=$(curl -H "Authorization: token ${GITHUB_TOKEN}" -X POST \
    -H "Accept: application/vnd.github.wyandotte-preview+json" \
    -d'{"lock_repositories":false,"repositories":['"${ALL_REPOS}"']}' \
    -sS https://api.github.com/orgs/${org}/migrations )

  RESPONSE_URL=$(echo ${RESPONSE})
  ERRORS=$(echo "${RESPONSE}" | grep '"message":')

  if [ -n "${ERRORS}" ]; then
   echo ${RESPONSE}
  else
    MIGRATION_URL=$(echo "${RESPONSE}" | grep url | grep migrations | sed -n 's/"url": "//p' | sed 's/.\{2\}$//')

# Send request to the migration starts endpoint to monitor the status of the export
  unset STATE
  until [[ ${STATE} == *"exported"* ]]
  do
    STATE="$(curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
    -H "Accept: application/vnd.github.wyandotte-preview+json" \
    ${MIGRATION_URL} \
    | grep -E '"state": ".*"')"
    echo ${STATE}
    sleep 5
  done

# Download the exported archive
  ARCHIVE_URL=`curl -H "Authorization: token ${GITHUB_TOKEN}" \
    -H "Accept: application/vnd.github.wyandotte-preview+json" \
    ${MIGRATION_URL}/archive`; \
    curl "${ARCHIVE_URL}" -o migration_archive.tar.gz
fi
