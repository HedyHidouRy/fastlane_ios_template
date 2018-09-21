# More documentation about how to customize your build
# can be found here:
# https://docs.fastlane.tools
fastlane_version "2.68.0"

default_platform :ios
 # Before starting anything, do those actions
  before_all do

  end

  ######################### PRIVATE LANES ##########################
  #--------------------- Building the project ---------------------#
  private_lane :build_release_version do |options|
    # Get or configure the needed certifcatitions
    cert(
      output_path: "fastlane/certificate",
      team_id: ENV["TEAM_ID"]
    )
    # Get the needed provision profiles
    sigh(
      output_path: "fastlane/certificate",
      team_id: ENV["TEAM_ID"]
    )
    # Build the iOS app, with enabling the code signing autocomatically
    automatic_code_signing(
      use_automatic_signing: true
    )
    # Make a build 
    gym(
      workspace: ENV["PROJECT_NAME"],
      scheme: ENV["RELEASE_SCHEME"],
      export_method: ENV["EXPORT_METHOD_APP_STORE"],
      silent: true,
      clean: true,
      export_xcargs: "-allowProvisioningUpdates",
      output_directory: "fastlane/ipa"
    )
  end
  #---------------------- Slack Message Sender --------------------#
  private_lane :to_slack do |options|
    project     = ENV["APP_NAME"]
    environment = options[:environment]
    destination = options[:destination]

    slack(
      message: " New :* #{project} iOS*  \n Running `#{environment}` build has been submitted to *#{destination}*  :rocket: \n _Sent from Fastlane_",
      default_payloads: [],
    )
  end
  ######################### PUBLIC LANES ###########################
  #--------------------- Register New Devices ---------------------#
  desc "Update the list of devices for existing provision profile"
  lane :register_new_devices do 
    register_devices(
      devices_file: ENV["DEVICES_FILE"]
      )
  end
  #--------------------- Install Dependencies ---------------------#
  desc "Install all dependencies"
  lane :install_deps do
    cocoapods
  end
  #--------------------- Build, Ship to Store ---------------------#
  desc "Add the build to App Store For verification"
  lane :to_appstore do
    # Build release version
    build_release_version
    # Deliver the build to App Store Connect
    deliver(
      submit_for_review: false,
      force: true,
      metadata_path: "./metadata",
      skip_screenshots: true
    )
     # Inform Slack
     to_slack(
      environment: "Production",
      destination: "App Store"
    )
  end
  #------------------- Build, Ship to TestFlight ------------------#
  desc "Add new build to TestFlight"
  lane :to_testflight do 
    # Build release version
    build_release_version
    # Pilot the build to TestFlight
    pilot(
      skip_submission: true,
      notify_external_testers: true,
      testers_file_path: ENV["TESTFLIGHT_TESTERS_FILE"]
    )
    # Inform Slack
    to_slack(
      environment: "Production",
      destination: "TestFlight"
    )
  end
  #----------------------- Build To Fabric -----------------------#
  desc "Add new build to Fabric, with possible option: `flavor: staging`"
  lane :ship_to_fabric do |options|
    # Configure with the passed flavor option
    flavor = ENV["RELEASE_SCHEME"]
    app_identifier = ENV["PROJECT_BUNDLE_ID"]
    notedFlavor = "Prod"
    if(options[:flavor]=='staging')
      flavor = ENV["PREPROD_SCHEME"]
      notedFlavor = "Staging"
      app_identifier = ENV["PROJECT_BUNDLE_ID_PREPROD"]
    end
    # Write change log with the right flavor
    changeLogFile = ENV['CHANGE_LOG_FILE']
    f = File.open(changeLogFile, "r+")
      changeLogLines = f.readlines
    f.close
    newChangeLog = ""
    if changeLogLines.empty?
      # If empty, show error to add some change log
      notification(
        subtitle: "No changelog",
        message:"Please write some changelog for the changes you've made",
        app_icon: ENV["APP_ICON"]
      )
      UI.user_error!("We are closing this, go write your change log")
    else
      if !changeLogLines[0].include? ENV["APP_NAME"]
        versionNb = get_version_number(target: flavor)
        changelogToAdd = ENV["APP_NAME"] + " " + notedFlavor + " " + versionNb + "\n"
        newChangeLog = [changelogToAdd] + changeLogLines
        output = File.new(changeLogFile, "w")
        newChangeLog.each { |line| output.write line }
        output.close
      end
    end
    # Get or configure the needed certifcatitions
    sigh(
      filename: ENV["PROJECT_NAME"]+".mobileprovision",
      output_path: "fastlane/certificate/",
      adhoc: true,
      team_id: ENV["TEAM_ID"],
      app_identifier: app_identifier
    )
    # Build the iOS app, with enabling the code signing autocomatically
    automatic_code_signing(
      use_automatic_signing: true
    )
    gym(
      workspace: ENV["PROJECT_NAME"],
      scheme: flavor,
      export_method: "ad-hoc",
      silent: true,
      clean: true,
      configuration: "Release",
      output_name: ENV["APP_NAME"],
      export_xcargs: "-allowProvisioningUpdates",
      suppress_xcode_output: true,
      output_directory: "fastlane/ipa"
    )
    # Get the list of emails
    emailsToInform = ""
    f = File.open(ENV["EMAILS_FILE"], "r+")
      emailsList = f.readlines
    f.close
    emailsList.each do |item|
      if emailsList[0] != item
        emailsToInform += ", "
      end 
      emailsToInform += item
    end
    puts "List of emails to inform " + emailsToInform
    # Upload to Beta by Crashlytics
    crashlytics(
      emails: emailsToInform,
      api_token: ENV["CRASHLYTICS_API_TOKEN"],
      build_secret: ENV["CRASHLYTICS_BUILD_SECRET"],
      notes_path: "fastlane/changelog.txt",
      ipa_path: "./fastlane/ipa/" + ENV["APP_NAME"] + ".ipa",
      debug: true
    )
    # Inform Slack
    to_slack(
      environment: notedFlavor,
      destination: "Fabric"
    )
    # After everything is finished, update change_log_archive
    File.open("build_notes.md", "a+") do |file|
      file.puts "\n"
      file.puts Time.now.inspect + " "
      file.puts newChangeLog
    end
    # Clean the tmp change log file
    File.open(changeLogFile, 'w') {|file| file.truncate(0) }

  after_all do |lane|
    notification(
      subtitle: "Building Finished",
      message:"Fastlane finished '#{lane}' successfully",
      app_icon: ENV["APP_ICON"]
    )
  end

  error do |lane, exception|
    notification(
      subtitle: "Building Failed",
      message:"Fastlane '#{lane}' errored",
      app_icon: ENV["APP_ICON"]
    )
  end

end
