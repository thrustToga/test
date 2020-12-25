fileprivate func startUpRequest() {

        let userDefaults = UserDefaults.standard
        
        let errorAlert = UIAlertController(title: "Error", message: "\nUnknownError", preferredStyle: .alert)
        let retryAction = UIAlertAction(title: "OK", style: .cancel)
        
        errorAlert.addAction(retryAction)
        
        LoadingView.buttonHandler = startUpRequest
        
        if let _ = userDefaults.string(forKey: "lastUpdate") {
            LoadingView.startLoadingView(loadingText: "Updating schedule")
            
            dateFormatter.timeZone = .current
            let dateNowString = dateFormatter.string(from: Date())
            dateFormatter.timeZone = TimeZone(identifier: "UTC")
            let dateNowOpt = dateFormatter.date(from: dateNowString)
            
            guard let lastDownloadScheduleUpdate = userDefaults.string(forKey: "lastDownloadScheduleUpdate") else {
                LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                    errorAlert.message = "\nData error!"
                    errorAlert.show()
                }
                
                return
            }
            guard let date = dateFormatter.date(from: lastDownloadScheduleUpdate) else {
                LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                    errorAlert.message = "\nDate format error!"
                    errorAlert.show()
                }
                
                return
            }
            guard let dateNow = dateNowOpt else {
                LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                    errorAlert.message = "\nDate format error!"
                    errorAlert.show()
                }
                
                return
            }
            guard let dateCompare = date.fullDistance(from: dateNow, resultIn: .day) else {
                LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                    errorAlert.message = "\nDate compare error!"
                    errorAlert.show()
                }
                
                return
            }
            
            if dateCompare >= 1 {
                JSONDecode.JSONScheduleLinksRequest { [unowned self] result in
                    switch result {
                    case .success(let scheduleLinks):
                        var facultyString = String()
                        switch userDefaults.integer(forKey: "faculty") {
                        case 0: facultyString = "ФАСК"
                        case 1: facultyString = "ФУВТ"
                        case 2: facultyString = "МФ"
                        case 3: facultyString = "ФПМиВТ"
                        default:
                            LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                                errorAlert.message = "\nData error!"
                                errorAlert.show()
                            }
                            
                            return
                        }
                        
                        guard let group = userDefaults.string(forKey: "group") else {
                            LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                                errorAlert.message = "\nData error!"
                                errorAlert.show()
                            }
                            
                            return
                        }
                        var groupExist = false
                        var groupURL = String()
                        
                        for facultyLink in scheduleLinks.faculties where facultyLink.faculty == facultyString {
                            for groupLink in facultyLink.groups where groupLink.group == group {
                                groupURL = groupLink.link
                                
                                groupExist = true
                            }
                        }
                        
                        if groupExist {
                            JSONDecode.JSONScheduleDataRequest(urlString: groupURL) { [unowned self] result in
                                switch result {
                                case .success(let scheduleData):
                                    DispatchQueue.main.async {
                                        if let error = RealmQueries.realmAddQuery(objects: scheduleData) {
                                            LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                                                errorAlert.message = "\nFailed to save schedule data\n\nError: \(error.localizedDescription)!"
                                                errorAlert.show()
                                            }
                                            
                                            return
                                        } else {
                                            if let error = RealmQueries.realmAddQuery(object: scheduleLinks) {
                                                LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                                                    errorAlert.message = "\nFailed to save schedule data\n\nError: \(error.localizedDescription)!"
                                                    errorAlert.show()
                                                }
                                                
                                                return
                                            } else {
                                                self.dateFormatter.timeZone = .current
                                                
                                                self.schedule = scheduleData
                                                
                                                userDefaults.set(scheduleLinks.lastUpdate, forKey: "lastUpdate")
                                                userDefaults.set(self.dateFormatter.string(from: Date()), forKey: "lastDownloadScheduleUpdate")
                                                
                                                userDefaults.removeObject(forKey: "lastTryUpdate")
                                                
                                                LoadingView.stopLoadingView(completionStyle: .success, endAnimationDuration: 1, resultText: "Successfully")
                                            }
                                        }
                                    }
                                    
                                case .failure(let error):
                                    LoadingView.stopLoadingView(completionStyle: .incomplete, resultText: "Incomplete!") {
                                        let incompleteAlert = UIAlertController(title: "Attention", message: "\(error.localizedDescription)\n\nLocal schedule data will be load!", preferredStyle: .alert)
                                        let incompleteAction = UIAlertAction(title: "OK", style: .cancel) { _ in
                                            guard self.schedule != nil else {
                                                LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                                                    errorAlert.message = "\nError to load schedule data!"
                                                    
                                                    errorAlert.show()
                                                }
                                                
                                                return
                                            }
                                            
                                            self.dateFormatter.timeZone = .current
                                            
                                            userDefaults.set(self.dateFormatter.string(from: Date()), forKey: "lastTryUpdate")
                                            
                                            LoadingView.incompleteRemoveLoadingView()
                                        }
                                        
                                        incompleteAlert.addAction(incompleteAction)
                                        
                                        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                                            incompleteAlert.show()
                                        }
                                    }
                                }
                            }
                        } else {
                            LoadingView.stopLoadingView(completionStyle: .incomplete, resultText: "Incomplete!") {
                                let incompleteAlert = UIAlertController(title: "Attention", message: "Group doesn't exist. Check available groups in settings.\n\nLocal schedule data will be load!", preferredStyle: .alert)
                                let incompleteAction = UIAlertAction(title: "OK", style: .cancel) { _ in
                                    guard self.schedule != nil else {
                                        LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                                            errorAlert.message = "\nError to load schedule data!"

                                            errorAlert.show()
                                        }

                                        return
                                    }

                                    self.dateFormatter.timeZone = .current

                                    userDefaults.set(self.dateFormatter.string(from: Date()), forKey: "lastTryUpdate")

                                    LoadingView.incompleteRemoveLoadingView()
                                }

                                incompleteAlert.addAction(incompleteAction)

                                DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                                    incompleteAlert.show()
                                }
                            }
                        }
                        
                    case .failure(let error):
                        LoadingView.stopLoadingView(completionStyle: .incomplete, resultText: "Incomplete!") {
                            let incompleteAlert = UIAlertController(title: "Attention", message: "\(error.localizedDescription)\n\nLocal schedule data will be load!", preferredStyle: .alert)
                            let incompleteAction = UIAlertAction(title: "OK", style: .cancel) { _ in
                                guard RealmData.shared.schedule != nil else {
                                    LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                                        errorAlert.message = "\nError to load schedule data!"

                                        errorAlert.show()
                                    }

                                    return
                                }

                                self.dateFormatter.timeZone = .current

                                userDefaults.set(self.dateFormatter.string(from: Date()), forKey: "lastTryUpdate")

                                LoadingView.incompleteRemoveLoadingView()
                            }

                            incompleteAlert.addAction(incompleteAction)

                            DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                                incompleteAlert.show()
                            }
                        }
                    }
                }
            } else {
                guard schedule != nil else {
                    LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                        errorAlert.message = "\nError to load schedule data!"
                        errorAlert.show()
                    }
                    
                    return
                }
                
                userDefaults.removeObject(forKey: "lastTryUpdate")
                
                LoadingView.stopLoadingView(completionStyle: .success, endAnimationDuration: 1, resultText: "Done!")
            }
        } else {
            if let newScheduleViewController = storyboard?.instantiateViewController(withIdentifier: "newScheduleViewController") as? NewScheduleViewController {
                
                newScheduleViewController.modalPresentationStyle = .fullScreen
                newScheduleViewController.modalTransitionStyle = .coverVertical
                
                self.newScheduleViewController = newScheduleViewController
            } else {
                fatalError("Error")
            }
            
            LoadingView.startLoadingView(loadingText: "Downloading schedule list")
            
            JSONDecode.JSONScheduleLinksRequest { [unowned self] result in
                switch result {
                case .success(let scheduleLinks):
                    self.newScheduleViewController?.scheduleLinks = scheduleLinks
                    
                    DispatchQueue.main.async {
                        self.present(self.newScheduleViewController!, animated: true)
                    }
                    LoadingView.stopLoadingView(completionStyle: .success, endAnimationDuration: 0.25, resultText: "Successfully")
                    
                case .failure(let error):
                    LoadingView.stopLoadingView(completionStyle: .failed, resultText: "Failed!") {
                        DispatchQueue.main.async {
                            errorAlert.message = "\n" + error.localizedDescription
                            errorAlert.show()
                        }
                    }
                }
            }
        }
    }
