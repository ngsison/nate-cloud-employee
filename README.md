# nate-cloud-employee

```
import Foundation

struct Item {
    
}

// Step 1: Create a protocol for the API service so that the ClassToTest would not have to depend on concrete implementations
protocol APIProtocol {
    func fetchItems(completion: (Result<[Item], Error>) -> Void)
}

// Step 2: Make API conform to APIProtocol so it can be passed as dependency of ClassToTest
class API: APIProtocol {
    func fetchItems(completion: (Result<[Item], Error>) -> Void) {
        // Fetch items from the real API
    }
}

// Step 3: Create a FakeAPI that conforms to APIProtocol so it can be passed ass dependency of ClassToTest during unit testing
class FakeAPI: APIProtocol {
    
    // Step 3.1: Make sure there's a flag to be able to predict and control the behavior of this FakeAPI
    private var returnSuccess: Bool
    init(returnSuccess: Bool) {
        self.returnSuccess = returnSuccess
    }
    
    // Step 3.2: The service should act based on the parameter `returnSuccess` that we injected when we initialized this FakeAPI
    func fetchItems(completion: (Result<[Item], Error>) -> Void) {
        if returnSuccess {
            let fakeItems = [Item(), Item(), Item()]
            completion(.success(fakeItems))
        } else {
            completion(.failure(NSError(domain: "someErrorDomain", code: 422)))
        }
    }
}

class ClassToTest {
    private(set) var stateList = [String]()
    
    // Step 4: Make sure that ClassToTest depends on an abstraction or protocol rather than concrete implementaion of a service
    private var apiService: APIProtocol
    init(apiService: APIProtocol) {
        self.apiService = apiService
    }

    func functionToTest() {
        stateList.append("Loading")

        apiService.fetchItems { result in
            switch result {
            case .success(_):
                stateList.append("Success")
            case .failure(_):
                stateList.append("Failed")
            }
        }
    }
}

// Step 5: Here are some examples how to perform tests using the FakeAPI service that we created

func testSuccessScenario() -> Bool {
    // arrange
    let fakeAPIService = FakeAPI(returnSuccess: true)
    let classToTest = ClassToTest(apiService: fakeAPIService)
    
    // act
    classToTest.functionToTest()
    
    // assert
    let testSuccess = classToTest.stateList.last == "Success"
    return testSuccess
}

func testFailedScenario() -> Bool {
    // arrange
    let fakeAPIService = FakeAPI(returnSuccess: false)
    let classToTest = ClassToTest(apiService: fakeAPIService)
    
    // act
    classToTest.functionToTest()
    
    // assert
    let testSuccess = classToTest.stateList.last == "Failed"
    return testSuccess
}

testSuccessScenario()
testFailedScenario()
```
